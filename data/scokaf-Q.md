# 1: Use of DelegateCall

Vulnerability details

### Impact

Delegatecall is difficult to utilize, and its application or misinterpretation might have disastrous effects.

Two considerations are necessary while utilizing delegatecall.

Context is preserved via delegatecall (storage, caller, etc...)
The contract calling delegatecall and the contract being called must have the same storage configuration.

## Proof of Concept

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L154  

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Use stateless Library


