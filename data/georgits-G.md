## Using `storage` instead of `memory` for structs/arrays saves gas
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L89
```
SubprotocolData memory subprotocolData = subprotocols[_name];
```
and then this line can be removed
```
subprotocols[_name] = subprotocolData;
```