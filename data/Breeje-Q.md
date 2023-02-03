## QA Report

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | EMITTING `CIDNFTRemoved` EVENT MISSING WHEN `register` OVERWRITES | 1 |
| [NC-2](#NC-2) | FIXING `REGISTER_FEE` CAN BE ISSUE IN LONG RUN | 1 |

### [NC-1] EMITTING `CIDNFTRemoved` EVENT MISSING WHEN `register` OVERWRITES

In `Register` method of `AddressRegistry`, when the registration already exists, then the method just overwrites and update the `cidNFTs` mapping. 

But while overwriting, there is no event emitted for Removal of Old Registration.

*Instance (1):*
```solidity
File: AddressRegistry.sol

    function register(uint256 _cidNFTID) external {
        if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
            // We only guarantee that a CID NFT is owned by the user at the time of registration
            // ownerOf reverts if non-existing ID is provided
            revert NFTNotOwnedByUser(_cidNFTID, msg.sender);
        cidNFTs[msg.sender] = _cidNFTID;
        emit CIDNFTAdded(msg.sender, _cidNFTID);
    }

```
[Link to Code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L42-L49)

#### Recommended Mitigation Step

Mitigated Code should be:

```solidity
File: AddressRegistry.sol

    function register(uint256 _cidNFTID) external {
        if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
            // We only guarantee that a CID NFT is owned by the user at the time of registration
            // ownerOf reverts if non-existing ID is provided
            revert NFTNotOwnedByUser(_cidNFTID, msg.sender);
        if(cidNFTs[msg.sender] != 0){
            emit CIDNFTRemoved(msg.sender, cidNFTID);
        }
        cidNFTs[msg.sender] = _cidNFTID;
        emit CIDNFTAdded(msg.sender, _cidNFTID);
    }

```
[Link to Code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L42-L49)

### [NC-2]  FIXING `REGISTER_FEE` CAN BE ISSUE IN LONG RUN

`REGISTER_FEE` has been fixed to `100 * 10**18` which is 100 $NOTE. But it's possible in future that the value of `NOTE` can be very high or very low. So it is recommended to have a function to change that value so that Registration Fees can be decided depending on the Value of `NOTE` at that time.