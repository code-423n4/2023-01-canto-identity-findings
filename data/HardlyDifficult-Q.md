## Low

### Consider allowing the subprotocol owner to be transferred

Once a subprotocol has been registered in the `SubprotocolRegistry` there is no way to update any of the values. The `SubprotocolData.owner` is the fee recipient leveraged as the fee recipient in [`CidNFT.add`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L193) - which means it may continue to collect fees overtime.

Since subprotocols are created by more than just the inner dev team, it may be more likely that the wallet becomes compromised or the subprotocol team desires switching to a multisig address in the future.

You could allow transferring the `owner` to another address, potentially with a 2-step process. This would not negatively impact use of the subprotocol, nor change the associated gas costs.

Similarly, consider allowing other fields to be updated. For example, maybe the subprotocol owner can lower the `fee` at any point but not increase it.

### Consider allowing fee wallet transfers

`cidFeeWallet` in `SubprotocolRegistry` and `CidNFT` are currently immutable addresses. Allowing the fee recipient to be updated may be appropriate for future proofing such as to allow moving the fee recipient under DAO control, or similar change.

You could allow transferring to another address, potentially with a 2-step process. However this would increase gas costs as immutable could no longer be used.

An alternative which preserves the current gas efficiency, is to use a simple contract as the wallet which escrows funds that can be withdrawn by its owner, which potentially uses [the OZ Ownable2Step mixin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol). The owner could be an EOA at first, and as the system matures it could be changed to a multisig or a DAO in the future.

### Consider a mutable baseURI

`baseURI` is assigned in the constructor with no way of updating it in the future. The approach used by [`tokenURI`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L140) prevents using IPFS since that would require knowing all potential tokenIds in advance. It may work with Arweave's append only file system, but the intended baseURI protocol is not clear from the docs or tests.

If these are to be hosted on a custom domain, it may be even more important that the `baseURI` is mutable, allowing it to be transitioned to a new store if that domain were to become unavailable (such as due to a security incident, trademark issue, or the host otherwise becoming unavailable in the future).

Alt: clarify in the docs the intended `baseURI`, to make it clear how permanence is guaranteed if that's part of the current launch plan.

### Consider requiring strings `.length != 0`

Several strings in the system are assigned once, without the ability to change it later on. Consider requiring that these are non-zero length to help ensure these have been assigned as expected.

* [SubprotocolRegistry.register](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L84) accepts a `_name` parameter of any length. Although only one entry could be registered with an empty name, it may be odd or unexpected to have a subprotocol with an empty string.
* [CidNFT.costructor](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L122) accepts a `_baseURI` parameter of any length which is then immutable. If this were not assigned then `tokenURI` would not work as expected.
* [`name` and `symbol` in `CidNFT.constructor`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L126) are allowed to be empty in the solmate constructor, may consider adding 0 length checks for these as well.

### Consider `isContract` checks in constructors

Several addresses are assigned in the contract constructors and assigned to immutable variables. A successful deployment is sensitive to these addresses being assigned correctly for the current network, and that addresses were specified in the correct order. Consider adding checks, as aggressively as possible for the use case, to help ensure the deployment configuration is correct.

* [`AddressRegistry.constructor`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L37) consider requiring that `_cidNFT.isContract()`.
* [`SubprotocolRegistry.constructor`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L66) and [`CidNFT.constructor`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L129) consider requiring that `_noteContract.isContract()`.
* [`CidNFT.constructor`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L66) consider requiring that `_subprotocolRegistry.isContract()`.

`.isContract()` is referring to the [OZ Address library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L40) or similar implementation.

### getActiveData could exceed gas limits

There's no upper bound on the size of the array which is being [returned in `CidNFT.getActiveData`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L319-L322). For both contract consumers and RPC requests, eventually a gas limit would be reached. Different RPC providers have different limits applied to view calls such as this.

Consider supporting a paginated variation where callers can request a subset of the full array when appropriate, as well as potentially a getter to get the length of the array.

## Non-Critical

### Simplify code

There is an if/else block in `CidNFT.add` for the `ACTIVE` type which could be simplified. Consider [changing this block of code](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L215-L226) into:

```solidity
// Check for duplicates
if (lengthBeforeAddition != 0 && activeData.positions[_nftIDToAdd] != 0) {
    revert ActiveArrayAlreadyContainsID(_cidNFTID, _subprotocolName, _nftIDToAdd);
}
activeData.values.push(_nftIDToAdd);
activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
```

This is easier to read and functionally the same. 

### Custom Error Params

Consider emitting the current owner in [`AddressRegistry.NFTNotOwnedByUser`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L46). `msg.sender` is included but that may already be clear from context. Including the owner could be a useful addition, e.g. allowing a user to quickly realize that the NFT is in another wallet of theirs.

### Emit from/to

Consider emitting the original CID in [`AddressRegistry.CIDNFTAdded`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L48) for when a user overwrites a prior registration. This may add a little gas overhead, but it should be minor since the slot is warm by setting the value here as well.

This is similar to [ERC-173 (the ownership standard)](https://eips.ethereum.org/EIPS/eip-173) which emits both the previous and new owner on transfer. Including both CIDs would make any overwrites more explicit for the user and any observing apps. When there is no previous CID, that param would emit 0 (similar to previous owner emitted as address(0) when first assigned).

Alternatively you could emit `CIDNFTRemoved` when there is a CID being overwritten. This approach would still offer the explicitness, but also be consistent with the `remove` flow, as if the user had made that call before the new `register` call.

### Consistent param ordering

Consider switching the param order in the `SubprotocolRegistered` event or the `register` function so that they are more consistent with each other. e.g. in the function, the order/active/primary bits come first while in the event they appear after the `_nftAddress`.

A possible improvement:

```solidity
    function register(
        string calldata _name,
        address _nftAddress,
        bool _ordered,
        bool _primary,
        bool _active,
        uint96 _fee
    ) external
```

The consistency improves readability, both for the code and for users reviewing transactions & events in a block explorer.

### Inconsistent used of named return params

Personally I prefer named return params, but it is a style preference. However in a project, it's nice to be consistent throughout with whichever style you prefer.

* [`SubprotocolRegister.getSubprotocol`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L106) does not name the return value while [`AddressRegistry.getCID`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L62) does, even though they are very similar functions.
* [`CidNFT.tokenURI`][https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L136] and [`CidNFT.onERC721Received`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L344) do not use named return params while all the other getters in this contract do.

### Use delete consistently

Consider using `delete` instead of [`= 0;` in `CidNFT.remove`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L284) similar to how delete is used [here](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L269) even though it is also just a `uint` being cleared. Other parts of the code also seem to use `delete` for similar cases.

### Spellcheck

* [existant -> existent](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L274)
* [subprotocl -> subprotocol](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L306) and [here](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L318)

Consider using the VSCode plugin `streetsidesoftware.code-spell-checker` or similar to help catch spelling errors during development.