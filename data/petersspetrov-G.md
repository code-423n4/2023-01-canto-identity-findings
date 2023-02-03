[G-01]USE A MORE RECENT VERSION OF SOLIDITY
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require()
 strings Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value
 
[G-02] USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT
 File: AddressRegistry.sol
  https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L4

 File: SubprotocolRegistry.sol
  https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L4-L7

 File: CidNFT.sol
   https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L5-L7

 Instead of this, use named imports as you do on for example;
  import {ERC721} from  "solmate/tokens/ERC721.sol";