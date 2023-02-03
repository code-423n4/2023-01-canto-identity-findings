## WRONG DOCUMENTATION/COMMENTS MIGHT CONFUSE USERS

### Description:

The comment here https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L16
says the fee (10%) is charged for every mint , this is incorrect , the correct statement is (should be) "CID_FEE_BPS is charged as a percentage of the subprotocol fee and only on adds (if the added subprotocol has a fee) . "
With the current description/comments this piece of code

```if (subprotocolFee != 0) { 
            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
            SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, cidFee);
            SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);
```
seems incorrect as it is only sending the fee when `subprotocolFee != 0` which is incorrect .

## INCORRECT DESCRIPTION OF THE PRIMARY ASSOCIATION

### Description:

The comment here https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L74 says primary association
is when a string key is mapped to a zero or one value , this is incorrect (as seen here https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L50 , primary is a uint) as primary is just one NFT ID.

## VIOLATION OF CIE PATTERN

###Description:

The pattern here https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L87-L94 violated the CIE pattern
as we are performing the transfer prior to the checks .
If $NOTE in future supports any hooks this might become a case for reentrancy too as there are no reentrancy guards too.

Remediation:

Perform the transfer after the checks.

## USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

### Description:

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).

Affected instances:
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L140
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L154 