## Subprotocol Fee may be 0 if set too low 
Line: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191

```
uint256 public constant CID_FEE_BPS = 1_000;
uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```

This can underflow when `subprotocolFee < 10` and be set to 0 which is not expected behavior by the user. 