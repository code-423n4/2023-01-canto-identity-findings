
## 01 Use scientific notation rather than exponentiation

Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. `10**18`)

Scientific notation should be used for better code readability

_There is 1 instance of this issue_

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol

```
File: src/SubprotocolRegistry.sol

17: uint256 public constant REGISTER_FEE = 100 * 10**18;
```

-----------

## 02 Use a more recent version of solidity

It's a best practice to use the latest compiler version.

The specified minimum compiler version is quite old. Older compilers might be susceptible to some bugs. We recommend changing the solidity version pragma to the latest version to enforce the use of an up to date compiler.

List of known compiler bugs and their severity can be found here: [https://etherscan.io/solcbuginfo](https://etherscan.io/solcbuginfo)

_This issue exists in all the In-scope contracts_. _There are 3 instances of this issue_

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol

```
File: src/CidNFT.sol

2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol

```
File: src/SubprotocolRegistry.sol

2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol

```
File: src/AddressRegistry.sol

2: pragma solidity >=0.8.0;
```

----------

## 03 Use named imports instead of plain 'import file.sol'

For instance, you use regular imports such as:

[AddressRegistry.sol#L4](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L4)

```
import "solmate/tokens/ERC721.sol";
```

Instead of this, use named imports such as:

```
import {ERC721.sol} from "solmate/tokens/ERC721.sol";
```

_There are 8 instances of this issue_

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol

```
File: src/CidNFT.sol

5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./SubprotocolRegistry.sol";
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol

```
File: src/SubprotocolRegistry.sol

4: import "solmate/tokens/ERC721.sol";
5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./CidSubprotocolNFT.sol";
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol

```
File: src/AddressRegistry.sol

4: import "solmate/tokens/ERC721.sol";
```

--------

## 04 Natspec is incomplete

_There are 2 instances of this issue_

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol

```
File: src/AddressRegistry.sol

42: function register(uint256 _cidNFTID) external {
52-53: function remove() external {
        uint256 cidNFTID = cidNFTs[msg.sender];
```

Missing: `@param _cidNFTID`, `@param cidNFTID`

----------

