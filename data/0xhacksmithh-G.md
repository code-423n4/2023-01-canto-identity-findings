### [Gas-01] Refactoring similar statements (Repetition of code present in CidNFT.sol contract file)
Following segment of code get repeated inside functions like ```add()``` and ```remove()``` 

Best way to enclose this code segment inside a private function and call it further inside add() and remove()
It will save some deployment costs.

```
SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
            _subprotocolName
        ); // @audit code repitation
        address subprotocolOwner = subprotocolData.owner;
        if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
        address cidNFTOwner = ownerOf[_cidNFTID];
        if (
            cidNFTOwner != msg.sender &&
            getApproved[_cidNFTID] != msg.sender &&
            !isApprovedForAll[cidNFTOwner][msg.sender]
        ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
```
*Instances(2)*
```solidity
File :: CidNFT.sol
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L172-L183
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L244-L254
```

### [Gas-02] Expression can be ```unchecked``` when overflow is not possible
*Instances(1)*
```solidity
File :: CidNFT.sol
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L150
```

### [Gas-03] Use ```bytes32``` instead of ```string```
Use bytes32 instead of string to save gas whenever possible. ```string``` is a dynamic data structure and therefore is more gas consuming than bytes32. 
*Instances(2)*
```solidity
File :: CidNFT.sol
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L70
```
```solidity
File :: SubprotocolRegistry.sol
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L40
```
