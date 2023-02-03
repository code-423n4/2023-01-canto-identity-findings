## Summary
### Low-Risk Issues
|ID|Title|Context|
|:--:|:-------|:--:|
|[L-01]| Missing zero address check | 1 |

Total issues: 1

### Non-Critical Issues
|ID|Title|Context|
|:--:|:-------|:--:|
|[N-01]| Use a modifier to avoid duplicate code to check the sender | 1 |
|[N-02]| Inconsistent function visibility order | 1 |
|[N-03]| Lock pragmas to a specific compiler version | 4 |
|[N-04]| Missing param in natspec in `AddressRegistry` | 1 |
|[N-05]| Missing explicit exports | 4 |
|[N-06]| Showing the full-length numbers in comments increase readability | 1 |

Total issues: 6

## Low Risk
### [L-01] Missing zero address check 

**Context**

```solidity
5 results - 3 files


src/AddressRegistry.sol:

37: cidNFT = _cidNFT;


src/CidNFT.sol:

128: cidFeeWallet = _cidFeeWallet;
129: note = ERC20(_noteContract);


src/SubprotocolRegistry.sol:

66: note = ERC20(_noteContract);
67: cidFeeWallet = _cidFeeWallet;

```

**Description**

It's important to do a zero address check for addresses to avoid wasting gas to redeploy the smart contract.

## Non-Critical

### [NC-01] Use a modifier to avoid duplicate code to check the sender

**Context**

```solidity
src/CidNFT.sol

177: address cidNFTOwner = ownerOf[_cidNFTID];
178: if (
179:     cidNFTOwner != msg.sender &&
180:     getApproved[_cidNFTID] != msg.sender &&
181:     !isApprovedForAll[cidNFTOwner][msg.sender]
182: ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);

247: address cidNFTOwner = ownerOf[_cidNFTID];
248: if (
249:     cidNFTOwner != msg.sender &&
250:     getApproved[_cidNFTID] != msg.sender &&
251:     !isApprovedForAll[cidNFTOwner][msg.sender]
252: ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
```

**Description**

There is some duplicated code in both `add` and `remove` function in `CidNFT.sol` to check if the sender is the owner of `_cidNFTID`. It would be better to use a modifier like `onlyCidNFTOwner` to avoid duplicate code and to improve readability.

### [NC-02] Inconsistent function visibility order

**Context**

`src/CidNFT.sol`

**Description**

Ordering the function in terms of visibility helps readers identify and study the contract functions, but there are contracts in the project that do not comply with this best practice.

Consider reordering the functions, by grouping them by visibility `[public, external, internal, private]`, and type `[pure, constant, view, payable]`.

### [NC-03] Lock pragmas to a specific compiler version

**Context**

```solidity
5 results - 4 files

src/AddressRegistry.sol:

2: pragma solidity >=0.8.0;


src/CidNFT.sol:

2: pragma solidity >=0.8.0;


src/CidSubprotocolNFT.sol:

2: pragma solidity >=0.8.0;


src/SubprotocolRegistry.sol:

2: pragma solidity >=0.8.0;

```

**Description:**
Pragma statements are appropriate when the contract is a library that is intended for consumption by other developers. Otherwise, the developer would need to manually update the pragma, in order to compile locally.

[As a best practice](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/) it's better to lock the pragma to a specific version.

### [NC-04] Missing param in natspec in `AddressRegistry`

**Context**

```solidity
src/AddressRegistry.sol

40: /// @notice Register a CID NFT to the address of the caller. NFT has to be owned by the caller
41: /// @dev Will overwrite existing registration if any exists
42: function register(uint256 _cidNFTID) external {

```

**Description**

The `register` function is missing the NAT parameter `_cidNFTID`


### [NC-05] Missing explicit exports
**Context**

```solidity

9 results - 4 files

src/SubprotocolRegistry.sol:

4: import "solmate/tokens/ERC721.sol";
5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./CidSubprotocolNFT.sol";


src/CidSubprotocolNFT.sol:

4: import "solmate/tokens/ERC721.sol";


src/CidNFT.sol:

5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./SubprotocolRegistry.sol";

src/AddressRegistry.sol

4: import "solmate/tokens/ERC721.sol";
```

**Description**

Instead of importing everything from a file, it's better to use the following notation to improve readability:

`import {Contract} from "contract.sol";`

### [NC-06] Showing the full-length numbers in comments increase readability

**Context**
```diff
src/SubprotocolRegistry.sol

- 17:      uint256 public constant REGISTER_FEE = 100 * 10**18;
+ 17:      uint256 public constant REGISTER_FEE = 100 * 10**18; // 100_000_000_000_000_000_000
```