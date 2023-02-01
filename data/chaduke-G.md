G1. https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L89-L90

Two gas savings here: 1) No need to cache the whole struct, since we only need to read the owner information; 2) we don't even need to cache the owner since we only need it ONCE. The revision is as follows.

```javascript
        SubprotocolData memory subprotocolData;  // @audit: only declare it no copy here
        if (subprotocols[_name].owner != address(0))  // @audit: only read it once, no caching necessary
                revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
        subprotocolData.owner = msg.sender;
```
