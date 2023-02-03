## QA Issues List
Issue 
## [N-01] For modern and more readable code; update import usages
## [N-02] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)
## [N-03] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS
Total: 13 contexts over 03 issues

## [N-01] For modern and more readable code; update import usages
Description:
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
Recommendation:
import {contract1 , contract2} from "filename.sol";
Contexts: 08
File: main/src/AddressRegistry.sol

    4: import "solmate/tokens/ERC721.sol";

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L4

File: main/src/SubprotocolRegistry.sol

    4: import "solmate/tokens/ERC721.sol";
    5: import "solmate/tokens/ERC20.sol";
    6: import "solmate/utils/SafeTransferLib.sol";
    7: import "./CidSubprotocolNFT.sol";

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L4-L7

File: main/src/CidNFT.sol

    5: import "solmate/tokens/ERC20.sol";
    6: import "solmate/utils/SafeTransferLib.sol";
    7: import "./SubprotocolRegistry.sol";

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L5-L7

## [N-02] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)
Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled
Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).
Contexts: 02
File: main/src/CidNFT.sol

    140:        return string(abi.encodePacked(baseURI, _id, ".json"));
    154:            ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L140
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L154

## [N-03] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS
Context: All Contracts
Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Recommendation:
NatSpec comments should be increased in contracts