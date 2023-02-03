## Summary

G-01 State variables only set in the constructor should be declared immutable | 1 instance

Total: 1 instance in 1 issue (20000 gas saved min)

---

## G-01 State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmacces (100 gas) with a PUSH32 (3 gas).

Instance(1):

CidNFT.sol

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L37

```solidity
    string public baseURI;
```