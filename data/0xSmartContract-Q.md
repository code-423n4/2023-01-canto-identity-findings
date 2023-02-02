## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]| No Check if OnErc721Received is implemented| 1 |
|[L-02]| Omissions in Events | 1 |
|[L-03]|The project has the risk of getting a Vampire Attack due to the fixed fee rate |  |
|[L-04]|Use the latest updated version of solmate dependencies| 1 |
|[L-05]| Add parameter to Event-Emit for critical function| 1 |
|[L-06]| Missing Event for  initialize | 3 |
|[L-07]|Insufficient coverage |  |
|[L-08]| Project Upgrade and Stop Scenario should be|  |
|[L-09]| Should an airdrop token arrive on the `CidNFT.sol` contract, it will be stuck|  |
|[L-10]| Protect your NFT from copying in fork|  |
|[L-11]| Using >/>= without specifying an upper bound is unsafe| 3 |
|[L-12]| The value `cidFee` is the revenue of the platform and should be rounded up instead of rounded down|1  |

Total 12 issues


### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [NC-01]|NatSpec comment should be add in `onERC721Received` function|1|
| [NC-02] |`Function writing` that does not comply with the `Solidity Style Guide`  |1|
| [NC-03] |Tokens accidentally sent to the contract (CifNFT.sol) cannot be recovered| 1 |
| [NC-04] |Take advantage of Custom Error's return value property | 1 |
| [NC-05] |Repeated validation logic can be converted into a function modifier | 4|
| [NC-06] |For modern and more readable code; update import usages  | 8 |
| [NC-07] |Use a more recent version of Solidity| All Contracts |
| [NC-08] |For functions, follow Solidity standard naming conventions (internal function style rule) | 1 |
| [NC-09] |Use SMTChecker|  |
| [NC-10] |Lines are too long| 5  |
| [NC-11] |Add EIP-2981 NFT Royalty Standart Support| |
| [NC-12] |Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists |2  |
| [NC-13] |No same value input control| 1 |

Total 13 issues

### [L-01] No Check if OnErc721Received is implemented

`CidNFT.mint()` will mint a NFT . When minting a NFT, the function does not check if a receiving contract implements onERC721Received().
`msg.sender` can be contract address.

The intention behind this function is to check if the address receiving the NFT, if it is a contract, implements onERC721Received(). Thus, there is no check whether the receiving address supports ERC-721 tokens and position could be not transferrable in some cases.

Following shows that _mint is used instead of _safeMint

```solidity

src/CidNFT.sol:
  146      /// The parameters should not include the function selector itself, the function select for add is always prepended.
  147:     function mint(bytes[] calldata _addList) external {
  148:         _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
  149:         bytes4 addSelector = this.add.selector; 
  150:         for (uint256 i = 0; i < _addList.length; ++i) {
  151:             (
  152:                 bool success, /*bytes memory result*/
  153: 
  154:             ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
  155:             if (!success) revert AddCallAfterMintingFailed(i);
  156:         }
  157:     }

```

Recommendation
Consider using _safeMint instead of _mint


### [L-02] Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The a lot of events should include the msg.sender

```diff

1 result - 1 file

src/CidNFT.sol:
  285              nftToRemove.safeTransferFrom(address(this), msg.sender, _nftIDToRemove);
-  286:             emit ActiveDataRemoved(_cidNFTID, _subprotocolName, _nftIDToRemove);
+ 286:           	emit ActiveDataRemoved(msg.sender,_cidNFTID, _subprotocolName, _nftIDToRemove);
  287          }


```

### [L-03] The project has the risk of getting a Vampire Attack due to the fixed fee rate.


```solidity
src/CidNFT.sol:
  15  
  16:     /// @notice Fee (in BPS) that is charged for every mint (as a percentage of the mint fee). Fixed at 10%.
  17:     uint256 public constant CID_FEE_BPS = 1_000;

```

What is a Vampire Attack in Crypto?
https://medium.com/@buraktahtacioglu/looksrare-vampire-attack-on-opensea-blockchain-roadmap-29bd51753a64
https://omerkeman.medium.com/what-is-a-vampire-attack-in-crypto-fdfc5e1fc5fc


### Recommended Mitigation Steps

Instead of a fixed fee rate, this problem can be solved in two ways;
1- Function is added so that the rate of Fee can be updated by an authorized owner
2- Core Contract can be made upgradable (A different architecture is required for this)
3- The commission rate should be fixed with a certain upper rate so that users will not have a trust problem.


Uniswap v3  solved this issue as follows;
```solidity
 /// @inheritdoc IUniswapV3PoolOwnerActions
    function setFeeProtocol(uint8 feeProtocol0, uint8 feeProtocol1) external override lock onlyFactoryOwner {
        require(
            (feeProtocol0 == 0 || (feeProtocol0 >= 4 && feeProtocol0 <= 10)) &&
                (feeProtocol1 == 0 || (feeProtocol1 >= 4 && feeProtocol1 <= 10))
        );
        uint8 feeProtocolOld = slot0.feeProtocol;
        slot0.feeProtocol = feeProtocol0 + (feeProtocol1 << 4);
        emit SetFeeProtocol(feeProtocolOld % 16, feeProtocolOld >> 4, feeProtocol0, feeProtocol1);
    }
```

NFTX solved this issue as follows;

https://github.com/NFTX-project/nftx-protocol-v2/blob/master/contracts/solidity/NFTXVaultFactoryUpgradeable.sol#L77-L97

```solidity
 function setFactoryFees(
        uint256 mintFee, 
        uint256 randomRedeemFee, 
        uint256 targetRedeemFee,
        uint256 randomSwapFee, 
        uint256 targetSwapFee
    ) public onlyOwner virtual override {
        require(mintFee <= 0.5 ether, "Cannot > 0.5 ether");
        require(randomRedeemFee <= 0.5 ether, "Cannot > 0.5 ether");
        require(targetRedeemFee <= 0.5 ether, "Cannot > 0.5 ether");
        require(randomSwapFee <= 0.5 ether, "Cannot > 0.5 ether");
        require(targetSwapFee <= 0.5 ether, "Cannot > 0.5 ether");

        factoryMintFee = uint64(mintFee);
        factoryRandomRedeemFee = uint64(randomRedeemFee);
        factoryTargetRedeemFee = uint64(targetRedeemFee);
        factoryRandomSwapFee = uint64(randomSwapFee);
        factoryTargetSwapFee = uint64(targetSwapFee);

        emit UpdateFactoryFees(mintFee, randomRedeemFee, targetRedeemFee, randomSwapFee, targetSwapFee);
    }

```

### [L-04] Use the latest updated version of solmate dependencies

```js
lib/solmate/package.json:
  1  {
  2:   "name": "@rari-capital/solmate",
  3:   "license": "AGPL-3.0-only",
  4:   "version": "6.2.0",
  5    "description": "Modern, opinionated and gas optimized building blocks for smart contract development.",

```

### Proof Of Concept


https://github.com/transmissions11/solmate/blob/main/package.json#L4

```js
 "name": "solmate",
  "license": "AGPL-3.0-only",
  "version": "6.7.0",

```
### Recommended Mitigation Steps
Upgrade `solmate` to version 6.7.0


### [L-05] Add parameter to Event-Emit for critical function


```diff

src/CidNFT.sol:
  146      /// The parameters should not include the function selector itself, the function select for add is always prepended.
  147:     function mint(bytes[] calldata _addList) external {
  148:          if (_addList.length > 250) revert MaxBlockGasLimitError(_addList.length); 
  149:         _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
  150:         bytes4 addSelector = this.add.selector; // 0x8cec460b
  151:         for (uint256 i = 0; i < _addList.length; ++i) {
  152:             (
  153:                 bool success, /*bytes memory result*/
  154: 
  155:             ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
  156:             if (!success) revert AddCallAfterMintingFailed(i);
  157:         }
+	emit(msg.sender, _addList.length, numMinted);
  158:     }

```

### [L-06] Missing Event for  initialize

**Context:**
```solidity

3 results - 3 files

src/AddressRegistry.sol:
  38      /// @param _cidNFT Address of the CID NFT contract
  39:     constructor(address _cidNFT) {
  40          cidNFT = _cidNFT;

src/CidNFT.sol:
  118      /// @param _subprotocolRegistry Address of the subprotocol registry
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


src/SubprotocolRegistry.sol:
  64      /// @param _cidFeeWallet Address of the wallet that receives the fees
  65:     constructor(address _noteContract, address _cidFeeWallet) {
  66          note = ERC20(_noteContract);

src/SubprotocolRegistry.sol:
  64      /// @param _cidFeeWallet Address of the wallet that receives the fees
  65:     constructor(address _noteContract, address _cidFeeWallet) {
  66:         note = ERC20(_noteContract);
  67:         cidFeeWallet = _cidFeeWallet;
  68:     }

```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit

### [L-07] Insufficient coverage

**Description:**
The test coverage rate of the project is 88%. Testing all functions is best practice in terms of security criteria.

```js

| File                              | % Lines          | % Statements     | % Branches     | % Funcs        |
|-----------------------------------|------------------|------------------|----------------|----------------|
| src/CidNFT.sol                    | 100.00% (87/87)  | 99.04% (103/104) | 88.00% (44/50) | 100.00% (9/9)  |

```
Due to its capacity, test coverage is expected to be 100%.

### [L-08] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol


### [L-09] Should an airdrop token arrive on the `CidNFT.sol` contract, it will be stuck


NFT project owners are given airdrops, but there is no function to withdraw incoming airdrop tokens, so airdrop tokens will be stuck in the contract.

A common method for airdrops is to collect airdrops with `claim`, so the `CifNFT.sol` contract can be considered upgradagable, adding a function to make `claim`.



### Recommended Mitigation Steps

Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```


### [L-10] Protect your NFT from copying in fork

Occasionally, forks can happen on different blockchains.
The project will operate on the Polygon network.Recently, a fork took place on the Ethereum network as well.

There may be forked versions of Blockchains, which could cause confusion and lead to scams as duplicated NFT assets enter the market, then it could result in duplication of non-fungible tokens (NFTs) and there's likely to be some level of confusion around which assets are 'official' or 'authentic.’

A forked Blockchains, chain will thus have duplicated deeds that point to the same tokenURI

About Replay Attack:
https://twitter.com/elerium115/status/1558471934924431363?s=20&t=RRheaYJwo-GmSnePwofgag

The most important part here is NFT's tokenURI detail. If the update I suggested below is not done, Duplicate NFTs will appear as a result, potentially leading to confusion and scams.

### Tools Used
Manual Code Review


### Recommended Mitigation Steps

```diff
src/CidNFT.sol:
  135      /// @return tokenURI The URI of the queried token (path to a JSON file)
  136:     function tokenURI(uint256 _id) public view override returns (string memory) {
+ 	 if (block.chainid != 1) revert wrongChain(); // Ethereum Chain ID : 1  - Polygon Chain ID : 137
  137:         if (ownerOf[_id] == address(0))
  138:             // According to ERC721, this revert for non-existing tokens is required
  139:             revert TokenNotMinted(_id);
  140:         return string(abi.encodePacked(baseURI, _id, ".json"));
  141:     }

```

### [L-11] Using >/>= without specifying an upper bound is unsafe


```solidity

3 results - 3 files

src/AddressRegistry.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity >=0.8.0;
  3  

src/CidNFT.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity >=0.8.0;
  3  

src/SubprotocolRegistry.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity >=0.8.0;

```


### [L-12]  The value `cidFee` is the revenue of the platform and should be rounded up instead of rounded down.

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L189-L194


The `cidFee` value is the revenue of the platform and should be rounded up, it is rounded down and therefore the platform loses money

```solidity

src/CidNFT.sol:
  188          // Charge fee (subprotocol & CID fee) if configured
  189:         uint96 subprotocolFee = subprotocolData.fee;
  190:         if (subprotocolFee != 0) {
  191:             uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
  192              SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, cidFee);

```

Full Code;

```solidity

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

        // The CID Protocol safeguards the NFTs of subprotocols. Note that these NFTs are usually pointers to other data / NFTs (e.g., to an image NFT for profile pictures)
        ERC721 nftToAdd = ERC721(subprotocolData.nftAddress);
        nftToAdd.safeTransferFrom(msg.sender, address(this), _nftIDToAdd);
        // Charge fee (subprotocol & CID fee) if configured
        uint96 subprotocolFee = subprotocolData.fee;
        if (subprotocolFee != 0) {
            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
            SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, cidFee);
            SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);
        }
        if (_type == AssociationType.ORDERED) {
            if (!subprotocolData.ordered) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
            if (cidData[_cidNFTID][_subprotocolName].ordered[_key] != 0) {
                // Remove to ensure that user gets NFT back
                remove(_cidNFTID, _subprotocolName, _key, 0, _type);
            }
            cidData[_cidNFTID][_subprotocolName].ordered[_key] = _nftIDToAdd;
            emit OrderedDataAdded(_cidNFTID, _subprotocolName, _key, _nftIDToAdd);
        } else if (_type == AssociationType.PRIMARY) {
            if (!subprotocolData.primary) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
            if (cidData[_cidNFTID][_subprotocolName].primary != 0) {
                // Remove to ensure that user gets NFT back
                remove(_cidNFTID, _subprotocolName, 0, 0, _type);
            }
            cidData[_cidNFTID][_subprotocolName].primary = _nftIDToAdd;
            emit PrimaryDataAdded(_cidNFTID, _subprotocolName, _nftIDToAdd);
        } else if (_type == AssociationType.ACTIVE) {
            if (!subprotocolData.active) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
            IndexedArray storage activeData = cidData[_cidNFTID][_subprotocolName].active;
            uint256 lengthBeforeAddition = activeData.values.length;
            if (lengthBeforeAddition == 0) {
                uint256[] memory nftIDsToAdd = new uint256[](1);
                nftIDsToAdd[0] = _nftIDToAdd;
                activeData.values = nftIDsToAdd;
                activeData.positions[_nftIDToAdd] = 1; // Array index + 1
            } else {
                // Check for duplicates
                if (activeData.positions[_nftIDToAdd] != 0)
                    revert ActiveArrayAlreadyContainsID(_cidNFTID, _subprotocolName, _nftIDToAdd);
                activeData.values.push(_nftIDToAdd);
                activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
            }
            emit ActiveDataAdded(_cidNFTID, _subprotocolName, _nftIDToAdd, lengthBeforeAddition);
        }
    }

```


### [N-01] NatSpec comment should be add in `onERC721Received` function
**Context:**


```solidity
src/CidNFT.sol:
  338  
  339:     function onERC721Received(
  340:         address, /*operator*/
  341:         address, /*from*/
  342:         uint256, /*id*/
  343:         bytes calldata /*data*/
  344:     ) external pure returns (bytes4) {
  345:         return ERC721TokenReceiver.onERC721Received.selector;
  346:     }
  347  }

```
**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
NatSpec comment should be add in function


### [N-02] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
`CifNFT.sol`

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

### [N-03] Tokens accidentally sent to the contract (CifNFT.sol) cannot be recovered


It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps

Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```

### [N-04] Take advantage of Custom Error's return value property

An important feature of Custom Error is that values such as address, tokenID, msg.value can 
be written inside the `()` sign, this kind of approach provides a serious advantage in 
debugging and examining the revert details of dapps such as tenderly.


```solidity
src/CidNFT.sol:
  183:         if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); 
```


### [N-05] Repeated validation logic can be converted into a function modifier

If a query or logic is repeated over many lines, using a modifier improves the readability and reusability of the code


```solidity

4 results - 2 files

src/CidNFT.sol:
  137:         if (ownerOf[_id] == address(0))

  176:         if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);

  248:         if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);

src/SubprotocolRegistry.sol:
  90:         if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);

```


### [N-06] For modern and more readable code; update import usages

**Context:**

```solidity
8 results - 3 files

src/AddressRegistry.sol:
  4: import "solmate/tokens/ERC721.sol";

src/CidNFT.sol:
  5: import "solmate/tokens/ERC20.sol";
  6: import "solmate/utils/SafeTransferLib.sol";
  7: import "./SubprotocolRegistry.sol";

src/SubprotocolRegistry.sol:
  4: import "solmate/tokens/ERC721.sol";
  5: import "solmate/tokens/ERC20.sol";
  6: import "solmate/utils/SafeTransferLib.sol";
  7: import "./CidSubprotocolNFT.sol";


```


**Description:**
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it. 
This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

**Recommendation:**
`import {contract1 , contract2} from "filename.sol";`

A good example from the ArtGobblers project;
```js
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```



### [N-07] Use a more recent version of Solidity

**Context:**
All contracts

**Description:**
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
https://github.com/ethereum/solidity/blob/develop/Changelog.md


**Recommendation:**
Old version of Solidity is used `(0.8.0)`, newer version can be used `(0.8.17)` 

### [N-08] For functions, follow Solidity standard naming conventions (internal function style rule)

**Context:**
```solidity

src/CidNFT.sol:
  70:     mapping(uint256 => mapping(string => SubprotocolData)) internal cidData;

```


**Description:**
The above codes don't follow Solidity's standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)



### [N-09] Use SMTChecker

The *highest* tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

### [N-10] Lines are too long

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length

Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

There are many examples, some of which are listed below;

[CidNFT.sol#L185](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L185)

[CidNFT.sol#L258](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L258)

[CidNFT.sol#L261](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L261)

[CidNFT.sol#L148](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L148)

[SubprotocolRegistry.sol#L78](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L78)


### [N-11] Add EIP-2981 NFT Royalty Standart Support

Consider adding EIP-2981 NFT Royalty Standard to the project

https://eips.ethereum.org/EIPS/eip-2981


Royalty (Copyright – EIP 2981):

Fixed % royalties: For example, 6% of all sales go back to artists
Declining royalties: There may be a continuous decline in sales based on time or any other variable.
Dynamic royalties : Varies over time or sales amount
Upgradeable royalties: Allows a legal entity or NFT owner to change any copyright
Incremental royalties: No royalties, for example when sold for less than $100
Managed royalties : Funds are owned by a DAO, imagine the recipient is a DAO treasury
Royalties to different people : Collectors and artists can even use royalties, not specific to a particular personality


### [N-12] Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists

Solmate's SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn't exist (yet).

This is stated in the Solmate library:
https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9


This problem does not appear in the project, but in case of msg.sender in contract (very low probability) this question may occur
This information should be included in project documents and NatSpec comments.

```solidity
2 results - 2 files

src/CidNFT.sol:
  6: import "solmate/utils/SafeTransferLib.sol";

src/SubprotocolRegistry.sol:
  6: import "solmate/utils/SafeTransferLib.sol";

```
### [N-13] No same value input control


```solidity

src/AddressRegistry.sol:
  41      /// @dev Will overwrite existing registration if any exists
  42:     function register(uint256 _cidNFTID) external {
  43:         if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
  44:             // We only guarantee that a CID NFT is owned by the user at the time of registration
  45:             // ownerOf reverts if non-existing ID is provided
  46:             revert NFTNotOwnedByUser(_cidNFTID, msg.sender);
  47:         cidNFTs[msg.sender] = _cidNFTID;
  48:         emit CIDNFTAdded(msg.sender, _cidNFTID);
  49:     }
```


