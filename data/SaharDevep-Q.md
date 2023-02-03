# Audit Report

## Summery 
[L01] TOKENURI() REVERTS FOR TOKENS THAT DON’T IMPLEMENT IERC20METADATA
[N01] USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’
[N02] USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18)
[N03] Event is not emited in the important operation
[N04] CONSTANTS SHOULD BE USED INSTEAD OF MAGIC NUMBERS

## Issues found

### [L01] TOKENURI() REVERTS FOR TOKENS THAT DON’T IMPLEMENT IERC20METADATA
Use safeDecimals() instead

#### Findings:
```
src\CidNFT.sol::136 => function tokenURI(uint256 _id) public view override returns (string memory) {
```

#### Tools used
Manual

### [N01] USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’

#### Findings:
```
src\AddressRegistry.sol::4 => import "solmate/tokens/ERC721.sol";
src\CidNFT.sol::4
src\CidNFT.sol::5
src\CidNFT.sol::6 
src\CidNFT.sol::7
src\SubprotocolRegistry.sol::4
src\SubprotocolRegistry.sol::5
src\SubprotocolRegistry.sol::6 
src\SubprotocolRegistry.sol::7
```

#### Tools used
Manual


### [N02] USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18)

#### Findings:
```
src\SubprotocolRegistry.sol::17 => uint256 public constant REGISTER_FEE = 100 * 10**18;

```

#### Tools used
Manual

### [N03] Event is not emited in the important operation

#### Findings:
```
src\CidNFT.sol::147 =>function mint(bytes[] calldata _addList) external {
```

#### Tools used
Manual

### [N04] CONSTANTS SHOULD BE USED INSTEAD OF MAGIC NUMBERS

#### Findings:
```
src\CidNFT.sol::191 => uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```

#### Tools used
Manual