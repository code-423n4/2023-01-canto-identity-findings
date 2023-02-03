## QA Report Summary
1. L-1: Unbounded Loops usage.
2. NC-1: Be more specific about imports.
3. NC-2: `abi.encodePacked` is used instead of `abi.encode`.
4. NC-3: Use Modifier For Better Readability And code/checks reuse.
5. NC-4: `error` defined but not used anywhere. 

Bonus: I-1: Make a separate file for the `errors`.

## Low-Risk Findings:
### L-1: Unbounded Loops usage.
Functions `mint` in [CidNFT.sol#L150](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L150) have an unbounded loop, which can cause DoS for the users. 
```
    function mint(bytes[] calldata _addList) external {
        _mint(msg.sender, ++numMinted); 
        bytes4 addSelector = this.add.selector;
        for (uint256 i = 0; i < _addList.length; ++i) {
            (
                bool success, /*bytes memory result*/

            ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
            if (!success) revert AddCallAfterMintingFailed(i);
        }
    }
```
Set an upper limit to avoid DoS conditions.

## Non-Critical Findings:
### NC-1: Be more specific about imports.
The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace.
Instead, the Solidity docs recommend specifying imported symbols explicitly.
```
SubprotocolRegistry.sol:
L4: import "solmate/tokens/ERC721.sol";
L5: import "solmate/tokens/ERC20.sol";
L6: import "solmate/utils/SafeTransferLib.sol";
L7: import "./CidSubprotocolNFT.sol";

AddressRegistry.sol:
L4: import "solmate/tokens/ERC721.sol";

L5: import "solmate/tokens/ERC20.sol";
L6: import "solmate/utils/SafeTransferLib.sol";
L7: import "./SubprotocolRegistry.sol";
```
Use specific imports syntax per [solidity docs](https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files
) recommendation.
There is only one import that is specified:
`import {ERC721, ERC721TokenReceiver} from "solmate/tokens/ERC721.sol";`
Use this good practice throughout the whole project.

### NC-2: `abi.encodePacked` is used instead of `abi.encode`.
Use of abi.encodePacked is safe, but unnecessary and not recommended. abi.encodePacked can result in hash collisions when used with two dynamic arguments (string/bytes).
There is also a discussion of removing abi.encodePacked from future versions of Solidity (Ethereum/solidity#11593), so using abi.encode now will ensure compatibility in the future as well.
Affected source code:
[CidNFT.sol#L140](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L140):
```
return string(abi.encodePacked(baseURI, _id, ".json"));
```
[CidNFT.sol#L154](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L154):
```
(bool success, /*bytes memory result*/) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
```

### NC-3: Use Modifier For Better Readability And Code Reuse.
In CidNFT.sol, the following check is used twice in two functions, [add](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L178) & [remove](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L250).
```
if (cidNFTOwner != msg.sender &&getApproved[_cidNFTID] != msg.sender
&& !isApprovedForAll[cidNFTOwner][msg.sender]) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
```
To improve readability and code reuse, a `NotAuthorizedForCIDNFT` modifier can be defined instead of making the code DRY.

### NC-4: `error` defined but not used anywhere. 
This [error](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L106) is defined in the CidNFT.sol but not used:
`error NotAuthorizedForSubprotocolNFT(address caller, uint256 subprotocolNFTID);`
Remove this unused [error](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L106)

## Informational/Suggestions:
### I-1: Make a separate file for the `errors`.
Defining errors in a separate file will improve the code readability and size.
A good example of this can be found [here](https://github.com/PodShip/podship-contracts/blob/main/contracts/PodShipErrors.sol).