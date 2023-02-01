## [YO NC-1] Event is missing `indexed` fields

### Handle
yosuke

## Vulnerability details
### Impact

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

### Proof of Concept
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L81
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L82
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L94
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L95


### Recommended Mitigation Steps

ex)

before

```solidity
event SetOwner(address pendingOwner);
```

after

```solidity
event SetOwner(address indexed pendingOwner);
```

## [YO NC-2] Constants should be defined rather than using magic numbers

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191

### Recommended Mitigation Steps