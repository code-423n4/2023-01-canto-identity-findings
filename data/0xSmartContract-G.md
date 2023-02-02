### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] | By assigning a name to the ``mapping`` parameter, directly achieve the gas optimization of the ``getter`` function |1 |
| [G-02] |Failure to check the zero address in the constructor causes the contract to be deployed again |3 |
| [G-03] |Earlier if check gas can be saved |1 |
| [G-04] |Use hardcode address instead ``address(this)``|5 |
| [G-05] |Using delete instead of setting `info` struct to 0 saves gas |1|
| [G-06] |Calculation should not be made at ``constant`` |1|
| [G-07] |Avoid contract existence checks by using solidity version 0.8.10 or later (600 gas) |6|
| [G-08] |Setting the constructor to ``payable`` (39 gas per instance) |3|
| [G-09] |Optimize names to save gas |All contracts|
| [G-10] |Use nested if and, avoid multiple check combinations|2|
| [G-11] |Use a more recent version of solidity|3|
| [G-12] |Use ERC721A instead ERC721 ||

Total 12 issues

### [G-01] By assigning a name to the ``mapping`` parameter, directly achieve the gas optimization of the ``getter`` function

solc 0.8.18 just dropped:
https://github.com/ethereum/solidity/releases/tag/v0.8.18

mostly bug fixes and internal stuffs, but there's a new feature for naming mapping params.
Reference: https://twitter.com/jtriley_eth/status/1620842058364878850?s=20&t=B4E79gqL7TwWKYxof1wsDQ


It is gas optimized in terms of reducing the code size.

1 result - 1 file:
```diff
src\AddressRegistry.sol:

-  2: pragma solidity >=0.8.0;
+     pragma solidity 0.8.18;


- 21:    mapping(address => uint256) private cidNFTs;
+        mapping(address _user => uint256) public getCID;


- 62:    function getCID(address _user) external view returns (uint256 cidNFTID) {
- 63:        cidNFTID = cidNFTs[_user];
- 64:    }
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L21
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L62_L64


### [G-02] Failure to check the zero address in the constructor causes the contract to be deployed again

Zero address control is not performed in the constructor in all 3 contracts within the scope of the audit. Bypassing this check could cause the contract to be deployed by mistakenly entering a zero address. In this case, the contract will need to be redeployed. This means extra gas consumption as contract deployment costs are high.

```solidity
src\AddressRegistry.sol:
  36:     constructor(address _cidNFT) {
  37:         cidNFT = _cidNFT;
  38:     }
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L36-L38

```solidity
src\CidNFT.sol:
  119:     constructor(
  120:         string memory _name,
  121:         string memory _symbol,
  122:         string memory _baseURI,
  123:         address _cidFeeWallet,
  124:         address _noteContract,
  125:         address _subprotocolRegistry
  126:     ) ERC721(_name, _symbol) {
  127:         baseURI = _baseURI;
  128:         cidFeeWallet = _cidFeeWallet;
  129:         note = ERC20(_noteContract);
  130:         subprotocolRegistry = SubprotocolRegistry(_subprotocolRegistry);
  131:     }
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L119-L131

```solidity
src\SubprotocolRegistry.sol:
  65:     constructor(address _noteContract, address _cidFeeWallet) {
  66:         note = ERC20(_noteContract);
  67:         cidFeeWallet = _cidFeeWallet;
  68:     }
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L65-L68


### [G-03] Earlier if check gas can be saved

In the ``register`` function, gas savings can be achieved by making the if check on line 93 earlier.

```diff
src\SubprotocolRegistry.sol:

  79:     function register(

  88:       if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);
  89:       SubprotocolData memory subprotocolData = subprotocols[_name]; 
  90:       if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
+ 93:       if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
+ 94:          revert NotASubprotocolNFT(_nftAddress);
  91:       subprotocolData.owner = msg.sender;
  92:       subprotocolData.fee = _fee;
- 93:       if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
- 94:           revert NotASubprotocolNFT(_nftAddress);
  95:       subprotocolData.nftAddress = _nftAddress;
  96:       subprotocolData.ordered = _ordered;
  97:       subprotocolData.primary = _primary;
  98:       subprotocolData.active = _active;
  99:       subprotocols[_name] = subprotocolData;
  100:      emit SubprotocolRegistered(msg.sender, _name, _nftAddress, _ordered, _primary, _active, _fee);
  101:     }
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L79-L101


### [G-04] Use hardcode address instead ``address(this)``

Instead of ``address(this)``, it is more gas-efficient to pre-calculate and use the address with a hardcode. Foundry's ``script.sol`` and solmate````LibRlp.sol`` contracts can do this.

Reference:
https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824


5 results - 1 file:
```solidity
src\CidNFT.sol:

  154:    ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
   
  187:    nftToAdd.safeTransferFrom(msg.sender, address(this), _nftIDToAdd);
 
  264:    nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
    
  270:    nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
   
  285:    nftToRemove.safeTransferFrom(address(this), msg.sender, _nftIDToRemove);
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L154


### [G-05] Using delete instead of setting `info` struct to 0 saves gas

```solidity
src\CidNFT.sol:

  284:             activeData.positions[_nftIDToRemove] = 0;
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L284


### [G-06] Calculation should not be made at ``constant``

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas. Calculation should not be done in ``constant``, writing directly saves some gas.
 
Note: It is not known whether the optimizer is turned on because the `foundry.toml` file does not exist. If the optimizer is on, there is gas saving.


```solidity
src\SubprotocolRegistry.sol:

 17:     uint256 public constant REGISTER_FEE = 100 * 10**18;
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17


### [G-07] Avoid contract existence checks by using solidity version 0.8.10 or later (600 gas)

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value

6 result - 3 files:
```solidity
src\AddressRegistry.sol:

  // @audit ownerOf()   
  43:     if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L43

```solidity
src\CidNFT.sol:

  // @audit safeTransferFrom()
  187:    nftToAdd.cmsg.sender, address(this), _nftIDToAdd);
 
  // @audit safeTransferFrom()
  264:    nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
 
   // @audit safeTransferFrom()
  270:    nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);

  // @audit safeTransferFrom()
  285:    nftToRemove.safeTransferFrom(address(this), msg.sender, _nftIDToRemove);
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L187

```solidity
src\SubprotocolRegistry.sol:

  // @audit supportsInterface()
  93:     if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L93


### [G-08] Setting the constructor to ``payable`` (39 gas per instance)

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.


3 results - 3 files:
```solidity
src\AddressRegistry.sol:

  36:     constructor(address _cidNFT) {
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L36

```solidity
src\CidNFT.sol:

  119:     constructor(
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L119



```solidity
src\SubprotocolRegistry.sol:

  65:     constructor(address _noteContract, address _cidFeeWallet) {
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L65



**Recommendation:**
Set the constructor to `payable`


### [G-09] Optimize names to save gas

**Context:** 
All Contracts

**Description:** 
Contracts most called functions could simply save gas by function ordering via ```Method ID```. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because ```22 gas``` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

**Recommendation:** 
Find a lower ```method ID``` name for the most called functions for example Call() vs. Call1() is cheaper by ```22 gas```
For example, the function IDs in the ``` CidNFT.sol ``` contract will be the most used; A lower method ID may be given.

**Proof of Consept:**
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

CidNFT.sol function names can be named and sorted according to METHOD ID

```js
Sighash   |   Function Signature
========================
82683154  =>  getOrderedData(uint256,string,uint256)
c87b56dd  =>  tokenURI(uint256)
16c464ae  =>  mint(bytes[])
d16c3577  =>  add(uint256,string,uint256,uint256,AssociationType)
0e339b4b  =>  remove(uint256,string,uint256,uint256,AssociationType)
816253ca  =>  getPrimaryData(uint256,string)
81bbb5bb  =>  getActiveData(uint256,string)
61dbb1d7  =>  activeDataIncludesNFT(uint256,string,uint256)
150b7a02  =>  onERC721Received(address,address,uint256,bytes)
```


### [G-10] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

2 results - 1 file:
```solidity
src\CidNFT.sol:

  178:         if (
  179:             cidNFTOwner != msg.sender &&
  180:             getApproved[_cidNFTID] != msg.sender &&
  181:             !isApprovedForAll[cidNFTOwner][msg.sender]
  182:         ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
  
  250:         if (
  251:             cidNFTOwner != msg.sender &&
  252:             getApproved[_cidNFTID] != msg.sender &&
  253:             !isApprovedForAll[cidNFTOwner][msg.sender]
  254:         ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L178-L182
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L250-L254


### [G-11] Use a more recent version of solidity

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

3 results - 3 files:
```solidity
src\AddressRegistry.sol:

  2: pragma solidity >=0.8.0;
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L2

```solidity
src\CidNFT.sol:

  2: pragma solidity >=0.8.0;
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L2

```solidity
src\SubprotocolRegistry.sol:

  2: pragma solidity >=0.8.0;
```

### [G-12] Use ERC721A instead ERC721

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee.

Reference: https://nextrope.com/erc721-vs-erc721a-2/