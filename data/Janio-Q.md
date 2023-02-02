### If the wallet that receives the fees is ever compromised there is no way to change the wallet


[`CidNFT.sol#L24`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L24)

The `cidFeeWallet` is immutable. Although it is set on the constructor, it can not be changed later on. 

```solidity
        address public immutable cidFeeWallet;
}
```
[`SubprotocolRegistry.sol#L23`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L23)
```solidity
    address public immutable cidFeeWallet;
}
```
Mitigating steps: remove immutable modifier and add a mechanism through which the wallet can be changed, like storing the deployers address and a function that only the deployer can execute to change the wallet address. Ownership also would be an alternative.
