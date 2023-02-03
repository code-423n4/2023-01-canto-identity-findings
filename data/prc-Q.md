## Summary

In the project code some percent basis points number literals were found making use of `_` separator as thousands separator.

## Findings

### Non-critical

- **Use of _ in percent bps number literals** - the use of `_` in numbers is a means of improving readability, and as the literals in the project are in percent basis points, the recommended placement would be like `10_00`, instead of the `1_000` that is used:

|`src/CidNFT.sol`|
|---|
|https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L17|
|https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L191|
