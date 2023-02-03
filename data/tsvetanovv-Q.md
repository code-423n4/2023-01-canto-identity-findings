### QA Issues List
| Number |Issues Details
|:--:|:-------|:--:|
|[QA-01]| Use latest Solidity version
|[QA-02]| Use stable pragma statement
|[QA-03]| Use named imports instead of plain `import File.sol`
|[QA-04]|Missing zero address validation
|[QA-05]|Use `_safeMint` instead of `_mint`
|[QA-06]| Constants should be defined rather than using magic numbers
|[QA-07]|Use scientific notation (E.G. 1e18) rather than exponentiation (E.G. 10**18)
|[QA-08]|Loss of precision due to rounding
|[QA-09]|Missing event for critical parameter change
***

## [QA-01] Use latest Solidity version

Solidity pragma versioning should be upgraded to latest available version - `0.8.18`. 
***

## [QA-02] Use stable pragma statement

Using a floating pragma statement `pragma solidity >=0.8.0` is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.
***

## [QA-03] USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT FILE.SOL`

In the following contracts this problem occurs:
```
src/CidNFT.sol
import "solmate/tokens/ERC20.sol";
import "solmate/utils/SafeTransferLib.sol";
import "./SubprotocolRegistry.sol";

SubprotocolRegistry.sol
import "solmate/tokens/ERC721.sol";
import "solmate/tokens/ERC20.sol";
import "solmate/utils/SafeTransferLib.sol";
import "./CidSubprotocolNFT.sol";

AddressRegistry.sol
import "solmate/tokens/ERC721.sol";
```
**Recommendation:**
`import {contract1 , contract2} from "filename.sol";`
***

## [QA-04] Missing zero address validation

Setters of address type parameters should include a zero-address check otherwise contract functionality may become inaccessible or tokens burnt forever.

```
CidNFT.sol
129: note = ERC20(_noteContract);
130: subprotocolRegistry = SubprotocolRegistry(_subprotocolRegistry);
```

#### Recommendation

Check that the address is not zero.
***

## [QA-05] Use `_safeMint` instead of `_mint`
In project documentation we see:
`// We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back`

But it is still advisable to use `_safeMint`
If msg.sender is a contract address that does not support ERC721, the NFT can be frozen in the contract.

### Recommendation
Use safeMint instead of mint to check received address support for ERC721 implementation.
If you are using `_safeMint`, add `nonReentrant` modifier to `safeMint()` function.
***

## [QA-06] Constants should be defined rather than using magic numbers
```
CidNFT.sol
191: uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```
***

## [N-07] USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18)
```
SubprotocolRegistry.sol
17: uint256 public constant REGISTER_FEE = 100 * 10**18;
```
***

## [QA-08] LOSS OF PRECISION DUE TO ROUNDING

```
CidNFT.sol
191: uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```
***

## [QA-09] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.
```
CidNFT.sol:
147: function mint(bytes[] calldata _addList) external {
```