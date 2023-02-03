
## [G-1] Wrong fields layout 
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L25-L37

Reorder fields layout to save storage and gas.
```solidity
    struct SubprotocolData {
        address owner;
        address nftAddress;
        uint96 fee;
        bool ordered;
        bool primary;
        bool active;
    }
```

----

## [G-2] Refactor CidNFT to save gas

rewrite function ```add```: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L159-L229 and ```remove```: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L231-L288

create private  functions ```addOrdered```, ```addPrimary```, ```addActive``` -- this functions should work with already validated fields, and don't do any additional checks.

create private functions ```removeOrdered```, ```removePrimary```, ```removeActive```  -- this functions should work with already validated fields too, and don't do any additional checks.

so, for example part with adding ordered record might be rewritten to:

```solidity
if (!subprotocolData.ordered) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
if (cidData[_cidNFTID][_subprotocolName].ordered[_key] != 0) {
    removeOrdered(_cidNFTID, _subprotocolName, _key);
}
cidData[_cidNFTID][_subprotocolName].ordered[_key] = _nftIDToAdd;
emit OrderedDataAdded(_cidNFTID, _subprotocolName, _key, _nftIDToAdd);
```

in the third line no additional checks will be performed and it saves a lot of gas and make code more readable.

