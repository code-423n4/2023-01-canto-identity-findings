### Move checks to the top

Checks, effects, interactions is a general best practice and can be applicable to more than just reentrancy concerns. When one of the following error scenarios applies, users pay gas for all statements executed up until the revert itself. By performing checks such as these as early as possible, you are saving users gas in failure scenarios without any sacrifice to the happy case costs. 

Moving the requirements to the top of the function can also improve readability.

Consider moving these checks to the top of the `register` function in `SubprotocolRegistry`:
* [`if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L88)
* [`if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L93-L94)

And potentially moving [`SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L87) below the check [`if (subprotocolData.owner != address(0))`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L90)

In `CidNFT` consider moving the check [`if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols();`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L183) to the top of the function.

### Revert on no-op

Unfortunately it can be common for users to accidentally fire the same transaction multiple times. This can result from an unclear app experience, or users misunderstanding speed up / replace transaction. When this occurs the duplicate transaction should be a no-op, ideally reverting as early as possible to limit gas costs. Reverting in the case of a duplicate may also prevent the redundant transaction from being broadcasted at all since apps and wallets typically estimate gas before broadcasting and that would show that it's expected to revert if the original has already been mined.

In `register` consider reverting if the provided `_cidNFTID` is already registered by that user. 

This is similar to the pattern already used [in `remove`, which reverts with `NoCIDNFTRegisteredForUser` when the call would otherwise be a no-op](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L54) (and this is not strictly necessary). 

### Remove helpers

[This block of code](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L244-L254) is redundant when passing from `add` -> `remove` which includes an external call to the `SubprotocolRegistry`. By moving the rest of the remove function into a private helper that's called by both `add` and `remove` you could save an extra call and therefor some gas in those scenarios, without complicating the code much.

This is related to the known gas optimization issue, but covers a redundant external call as well as the redundant sloads for approval checking.

Additionally you could use more targeted helpers. As a POC, here's the impact from extracting a `_removeOrdered` helper:

```
original:
[PASS] testOverwritingOrdered() (gas: 281662)

new:
[PASS] testOverwritingOrdered() (gas: 277991)

Savings: 3,671
```

Using the following helper which is called in `add` instead of the `remove` currently there. And this helper can be shared with `remove` so that the logic is not repeated in the contract.

```solidity
function _removeOrdered(
    ERC721 nftToRemove,
    uint256 _cidNFTID,
    string calldata _subprotocolName,
    uint256 _key,
    uint256 _nftIDToRemove
) internal {
    delete cidData[_cidNFTID][_subprotocolName].ordered[_key];
    nftToRemove.safeTransferFrom(address(this), msg.sender, _nftIDToRemove);
    emit OrderedDataRemoved(_cidNFTID, _subprotocolName, _key, _nftIDToRemove);
}
```

### Storage

Consider a storage reference in `SubprotocolRegistry` for [`SubprotocolData memory subprotocolData`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L89). This would be cheaper in the case of `SubprotocolAlreadyExists` since that only accesses one of the two slots. And does not negatively impact the happy case. 

It may be considered cleaner syntax since [`subprotocols[_name] = subprotocolData;`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L99) may then be removed, but that is subjective.

### Unchecked when clearly safe to do so

Using `unchecked` blocks saves just a tiny bit of gas, but in instances where its clearly safe already it's possible to avoid this unnecessary check.

It's becoming a common pattern to use in `for` loops such as [this one in `CidNFT`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L150) where you could consider using:

```solidity
for (uint256 i = 0; i < _addList.length; ) {
    // existing logic
    unchecked {
        ++i;
    }
}
```

In `CidNFT` when calculating fees the math is using a constant, so `cidFee` is always less than `subprotocolFee` making the subtraction always safe when calculating [`subprotocolFee - cidFee`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L193).

In `CidNFT` [`arrayLength - 1`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L279-L280) is guaranteed to be safe since there is a check [`if (arrayPosition == 0) revert...`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L275) above which handles the potential underflow scenario already.