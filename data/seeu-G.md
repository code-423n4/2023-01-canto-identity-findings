## [G-01] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

### Description

Gas consumption can be greater if you use items that are less than 32 bytes in size. This is such that the EVM can only handle 32 bytes at once. In order to increase the element's size from 32 bytes to the necessary amount, the EVM must do extra operations if it is lower than that. When necessary, it is advised to utilize a bigger size and then downcast.

### Findings

- [src/CidNFT.sol#L189](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L189)
  ```Solidity
  uint96 subprotocolFee = subprotocolData.fee;
  ```

### References

- [Layout of State Variables in Storage | Solidity docs](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html#layout-of-state-variables-in-storage)
- [GAS OPTIMIZATIONS ISSUES by Gokberk Gulgun](https://hackmd.io/@W1m6lTsFT5WAy9C_lRTX_g/rkr5Laoys)