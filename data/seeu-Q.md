## [L-01]Outdated Compiler Version

### Description

Using an older compiler version might be risky, especially if the version in question has faults and problems that have been made public.

### Findings

- [src/AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol) => `>=0.8.0`
- [src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol) => `>=0.8.0`
- [src/SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol) => `>=0.8.0`

### References

- [SWC-102](https://swcregistry.io/docs/SWC-102)
- [Etherscan Solidity Bug Info](https://etherscan.io/solcbuginfo)


## [L-02] For modern and more readable code; update import usages

### Description

A less obvious way that solidity code is clearer is the struct Point. Prior to now, we imported it via global import, but we didn't use it. The Point struct contaminated the source code with an extra object that was not needed and that we were not utilizing.

To be sure to only import what you need, use specific imports using curly brackets.

### Findings

- [src/AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol)
  ```Solidity
  ::4 => import "solmate/tokens/ERC721.sol";
  ```
- [src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)
  ```Solidity
  ::5 => import "solmate/tokens/ERC20.sol";
  ::6 => import "solmate/utils/SafeTransferLib.sol";
  ::7 => import "./SubprotocolRegistry.sol";
  ```
- [src/SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)
  ```Solidity
  ::4 => import "solmate/tokens/ERC721.sol";
  ::5 => import "solmate/tokens/ERC20.sol";
  ::6 => import "solmate/utils/SafeTransferLib.sol";
  ::7 => import "./CidSubprotocolNFT.sol";
  ```

### References

- [[N-03] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES | PoolTogether contest](https://code4rena.com/reports/2022-12-pooltogether#n-03-for-modern-and-more-readable-code-update-import-usages)


## [L-03] Unnamed return parameters

### Description

To increase explicitness and readability, take into account introducing and utilizing named return parameters.

### Findings

- [src/AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol)
  ```Solidity
  ::62 =>     function getCID(address _user) external view returns (uint256 cidNFTID) {
  ```
- [src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)
  ```Solidity
  ::136 =>     function tokenURI(uint256 _id) public view override returns (string memory) {
  ```
- [src/SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)
  ```Solidity
  ::106 =>     function getSubprotocol(string calldata _name) external view returns (SubprotocolData memory) {
  ```

### References

- [Unnamed return parameters | Opyn Bull Strategy Contracts Audit](https://blog.openzeppelin.com/opyn-bull-strategy-contracts-audit/#unnamed-return-parameters)