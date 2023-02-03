# [L-01] Could add duplicate for `AssociationType.ORDERED`
## Targets
- https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L165-L229
## Impact
Let there be a subprotocol NFT, which can be retrieved from cidNFT. This could then be added back under a different key when using AssociationType.ORDERED, because there is no duplicate check like for AssociationType.ACTIVE.  
## Proof of Concept
There is a [test case](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/test/CidNFT.t.sol#L251-L273), which covers reverting when adding duplicates for AssociationType.ACTIVE, but not for AssociationType.ORDERED. We can check that it is indeed possible to add a duplicate for AssociationType.ORDERED when using the same assumptions as the test case above.

```solidity
   function testAddDuplicateOrdered() public {
        address user = user1;
        (uint256 tokenId, uint256 subId, uint256 key1) = prepareAddOne(user, user);
        uint256 key2 = 2;
        // Add Once
        vm.startPrank(user);
        cidNFT.add(tokenId, "sub1", key1, subId, CidNFT.AssociationType.ORDERED);
        vm.stopPrank();

        // Transfer the sub nft back to the user, this should not happen normally
        // since the sub nft should stay in the cid nft unless it got removed
        // but the SubprotocolNFT can have arbitrary logic e.g. admin right
        vm.startPrank(address(cidNFT));
        sub1.safeTransferFrom(address(cidNFT), user, subId);
        vm.stopPrank();

        // Add Twice. It won't revert!
        vm.startPrank(user);
        sub1.approve(address(cidNFT), subId);
        cidNFT.add(tokenId, "sub1", key2, subId, CidNFT.AssociationType.ORDERED);
        assertEq(cidNFT.getOrderedData(tokenId, "sub1", key1), subId);
        assertEq(cidNFT.getOrderedData(tokenId, "sub1", key2), subId);
        vm.stopPrank();
    }}
```

## Tools Used
VS Code, Manual review, Foundry
## Recommended Mitigation Steps
Consider tracking added subprotocol NFTs in a mapping and revert when adding a duplicate. Like it is done for the AssociationType.ACTIVE.
