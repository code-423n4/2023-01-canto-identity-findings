# 1. event SubprotocolRegistered should not mark string variable as indexed

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L45-L53

> However, string can have an arbitrary length. To still be able to store those values in a topic, Solidity creates a hash of the value, which ends up in the topic.
> The upside is that filters can still work, since you can just create the hash of the value you're filtering for. The downside is that web3 is unable to decode the value, since that would require performing the hash function in reverse direction.

ref: https://ethereum.stackexchange.com/questions/6840/indexed-event-with-string-not-getting-logged

String variable should not be marked as `indexed` in event, as it will be hashed in topics.

Suggestion: removed `indexed` keyword, change to:

```
    event SubprotocolRegistered(
        address indexed registrar,
        string name,
        address indexed nftAddress,
        bool ordered,
        bool primary,
        bool active,
        uint96 fee
    );
```

----

# 2. user will lost his _nftIDToAdd if wrong _type is passed to add function

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L165

`add` function is missing a last `else` branch to revert if `_type` parameter is not valid (not in ORDERED, PRIMARY and ACTIVE)

In this case, NFT is transfered to the CidNFT contract, and fee paied (if subprotocolFee>0), while cidData do not get updated.

Suggestion: add an else branch like:

```
if (_type == AssociationType.ORDERED) {
   ...
} else if (_type == AssociationType.PRIMARY) {
   ...
} else if (_type == AssociationType.ACTIVE) {
   ...
} else{
   revert TypeInvalid(_type);
}
```

and add an error:

```
error TypeInvalid(AssociationType type);
```

----

# 3. remove _nftIDToRemove from event OrderedDataRemoved and PrimaryDataRemoved

As the docs said _nftIDToRemove is only used in AssociationType ACTIVE
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L235


So there's no need to emit event including `_nftIDToRemove` parameter for ORDERED and PRIMARY

Suggestion: to avoid misunderstanding, change to:

```
    event OrderedDataRemoved(
        uint256 indexed cidNFTID,
        string indexed subprotocolName,
        uint256 indexed key
    );
    event PrimaryDataRemoved(uint256 indexed cidNFTID, string indexed subprotocolName);
```

