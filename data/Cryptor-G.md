## remove 0 to save gas 

Instead of using uint i = 0 as shown here 

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L150

use 

```

(uint i; i < _addList.length; ++i)

```