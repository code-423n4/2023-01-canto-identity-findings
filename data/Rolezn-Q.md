## Summary<a name="Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Add to `blacklist` function | 8 |
| [LOW&#x2011;2](#LOW&#x2011;2) | Possible rounding issue | 1 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Missing parameter validation in `constructor` | 6 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Protect your NFT from copying in POW forks | 1 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists | 2 |

Total: 18 contexts over 56 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Function writing that does not comply with the Solidity Style Guide  | 3 |
| [NC&#x2011;2](#NC&#x2011;2) | Use `delete` to Clear Variables | 1 |
| [NC&#x2011;3](#NC&#x2011;3) | NatSpec return parameters should be included in contracts | 1 |
| [NC&#x2011;4](#NC&#x2011;4) | Lines are too long | 2 |
| [NC&#x2011;5](#NC&#x2011;5) | Implementation contract may not be initialized | 3 |
| [NC&#x2011;6](#NC&#x2011;6) | NatSpec comments should be increased in contracts | 1 |
| [NC&#x2011;7](#NC&#x2011;7) | Non-usage of specific imports | 2 |
| [NC&#x2011;8](#NC&#x2011;8) | Use a more recent version of Solidity | 3 |
| [NC&#x2011;9](#NC&#x2011;9) | Use `bytes.concat()` | 2 |
| [NC&#x2011;10](#NC&#x2011;10) | Use standard decimals interpretations using `1eX`  | 1 |

Total: 19 contexts over 10 issues

## Low Risk Issues

### <a href="#Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Add to `blacklist` function

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```


#### <ins>Recommended Mitigation Steps</ins>
Add to Blacklist function and modifier.



### <a href="#Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Possible rounding issue

There might be a rounding issue as `CID_FEE_BPS` = 1000, however there is no defined value for `subprotocolFee` and it could be 0 or uint96.max. But values between 1-9 for `subprotocolFee` will result in `cidFee` being 0 due to rounding issue.

#### <ins>Proof Of Concept</ins>


```solidity
191: uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L191



### <a href="#Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Missing parameter validation in `constructor`

Some parameters of constructors are not checked for invalid values.

#### <ins>Proof Of Concept</ins>

```solidity
36: address _cidNFT

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol#L36

```solidity
123: address _cidFeeWallet
124: address _noteContract
125: address _subprotocolRegistry

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L123-L125

```solidity
65: address _noteContract
65: address _cidFeeWallet

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L65-L65



#### <ins>Recommended Mitigation Steps</ins>

Validate the parameters.





### <a href="#Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Protect your NFT from copying in POW forks
Ethereum has performed the long-awaited "merge" that will dramatically reduce the environmental impact of the network

There may be forked versions of Ethereum, which could cause confusion and lead to scams as duplicated NFT assets enter the market.

If the Ethereum Merge, which took place in September 2022, results in the Blockchain splitting into two Blockchains due to the 'THE DAO' attack in 2016, this could result in duplication of immutable tokens (NFTs).

In any case, duplicate NFTs will exist due to the ETH proof-of-work chain and other potential forks, and there’s likely to be some level of confusion around which assets are 'official' or 'authentic.'

Even so, there could be a frenzy for these copies, as NFT owners attempt to flip the proof-of-work versions of their valuable tokens.

As ETHPOW and any other forks spin off of the Ethereum mainnet, they will yield duplicate versions of Ethereum’s NFTs. An NFT is simply a blockchain token, and it can work as a deed of ownership to digital items like artwork and collectibles. A forked Ethereum chain will thus have duplicated deeds that point to the same tokenURI

About Merge Replay Attack: https://twitter.com/elerium115/status/1558471934924431363?s=20&t=RRheaYJwo-GmSnePwofgag

#### <ins>Proof Of Concept</ins>


```solidity
136: function tokenURI(uint256 _id) public view override returns (string memory) {

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L136



#### <ins>Recommended Mitigation Steps</ins>

Add the following check:
```solidity
if(block.chainid != 1) { 
    revert(); 
}
```



### <a href="#Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists

Solmate's SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn't exist (yet).

This is stated in the Solmate library: https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

#### <ins>Proof Of Concept</ins>


```solidity
6: import "solmate/utils/SafeTransferLib.sol";

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L6

```solidity
6: import "solmate/utils/SafeTransferLib.sol";

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L6




#### <ins>Recommended Mitigation Steps</ins>
Add a contract exist control in functions;

```solidity
pragma solidity >=0.8.0;

function isContract(address _addr) private returns (bool isContract) {
    isContract = _addr.code.length > 0;
}
```



## Non Critical Issues

### <a href="#Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Function writing that does not comply with the Solidity Style Guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

#### <ins>Proof Of Concept</ins>

All in-scope contracts




### <a href="#Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Use `delete` to Clear Variables

`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at `x`.

The `delete` key better conveys the intention and is also more idiomatic. Consider replacing assignments of zero with `delete` statements.

#### <ins>Proof Of Concept</ins>

```solidity
284: activeData.positions[_nftIDToRemove] = 0;

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L284








### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> NatSpec return parameters should be included in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
#### <ins>Proof Of Concept</ins>


All in-scope contracts


#### <ins>Recommended Mitigation Steps</ins>

Include return parameters in NatSpec comments

Recommendation Code Style: (from Uniswap3)

```solidity
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```

i.e.
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
https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L339




### <a href="#Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Lines are too long

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length
Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### <ins>Proof Of Concept</ins>

```solidity
185: // The CID Protocol safeguards the NFTs of subprotocols. Note that these NFTs are usually pointers to other data / NFTs (e.g., to an image NFT for profile pictures)
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L185

```solidity
78: /// @param _fee Fee (in $NOTE) for minting a new token of the subprotocol. Set to 0 if there is no fee. 10% is subtracted from this fee as a CID fee
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L78





### <a href="#Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Implementation contract may not be initialized

OpenZeppelin recommends that the initializer modifier be applied to constructors. 
Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.
https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### <ins>Proof Of Concept</ins>


```solidity
36: constructor(address _cidNFT)
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol#L36

```solidity
119: constructor(
        string memory _name,
        string memory _symbol,
        string memory _baseURI,
        address _cidFeeWallet,
        address _noteContract,
        address _subprotocolRegistry
    ) ERC721(_name, _symbol)
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L119

```solidity
65: constructor(address _noteContract, address _cidFeeWallet)
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L65





### <a href="#Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> NatSpec comments should be increased in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

#### <ins>Proof Of Concept</ins>


All in-scope contracts

i.e.
Natspec documentation missing info which includes about `@param _cidNFTID`

```solidity
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
https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol#L2

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
https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L339

#### <ins>Recommended Mitigation Steps</ins>

NatSpec comments should be increased in contracts



### <a href="#Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Non-usage of specific imports

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace.
Instead, the Solidity docs recommend specifying imported symbols explicitly.
https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

A good example:
```solidity
import {OwnableUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/OwnableUpgradeable.sol";
import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
import {SafeCastLib} from "solmate/utils/SafeCastLib.sol";
import {ERC20} from "solmate/tokens/ERC20.sol";
import {IProducer} from "src/interfaces/IProducer.sol";
import {GlobalState, UserState} from "src/Common.sol";
```

#### <ins>Proof Of Concept</ins>


```solidity
7: import "./SubprotocolRegistry.sol";

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L7

```solidity
7: import "./CidSubprotocolNFT.sol";

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L7




#### <ins>Recommended Mitigation Steps</ins>

Use specific imports syntax per solidity docs recommendation.



### <a href="#Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Use a more recent version of Solidity

<a href="https://blog.soliditylang.org/2021/04/21/solidity-0.8.4-release-announcement/">0.8.4</a>:
bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>)

<a href="https://blog.soliditylang.org/2022/02/16/solidity-0.8.12-release-announcement/">0.8.12</a>: 
string.concat() instead of abi.encodePacked(<str>,<str>)

<a href="https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/">0.8.13</a>: 
Ability to use using for with a list of free functions

<a href="https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/">0.8.14</a>:

ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.

<a href="https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/">0.8.15</a>:

Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.

<a href="https://blog.soliditylang.org/2022/08/08/solidity-0.8.16-release-announcement/">0.8.16</a>:

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

<a href="https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/">0.8.17</a>:

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L2



#### <ins>Recommended Mitigation Steps</ins>

Consider updating to a more recent solidity version.



### <a href="#Summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Use `bytes.concat()`

Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

#### <ins>Proof Of Concept</ins>


```solidity
140: return string(abi.encodePacked(baseURI, _id, ".json")

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L140






#### <ins>Recommended Mitigation Steps</ins>

Use `bytes.concat()` and upgrade to at least Solidity version 0.8.4 if required. 



### <a href="#Summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Use standard decimals interpretations using `1eX` 

Use 1eX instead of 1**X

#### <ins>Proof Of Concept</ins>

```solidity
17: uint256 public constant REGISTER_FEE = 100 * 10**18;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L17

#### <ins>Recommended Mitigation Steps</ins>

Replace 10**X with 1eX





