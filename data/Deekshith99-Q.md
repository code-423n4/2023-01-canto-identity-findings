# [QA Report]

## [L-1] SOLMATE’S `SAFETRANSFERLIB` DOESN’T CHECK WHETHER THE ERC20 CONTRACT EXISTS

Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet)
Refer this permalink 
https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9
There are 2 instances of it  
we have used in the following contracts [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol) & [SubprotocolRegistry.sol#L6](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol)

Reference was taken from [here](https://code4rena.com/reports/2022-12-caviar/#l-04-solmates-safetransferlib-doesnt-check-whether-the-erc20-contract-exists)

## [NC-1] Use a more recent version of solidity
context: All contracts ( currently we are using only >=0.8.0; )
Use a solidity version of at least 0.8.4 to get bytes.concat() instead of abi.encodePacked(,) Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked(,) Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions

## [NC-2] Missing Natspec 
There is 1 instance of it
[CidNFT.sol#L339](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L339)
its recommend to write Natspec comment to the `onERC721Received` function

