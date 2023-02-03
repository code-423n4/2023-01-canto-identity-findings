# CID fee may be withheld from protocol

## Impact
The `subprotocolOwner` can bypass the protocol's 10% CID fee by setting an extremely low `_fee` during registration. The impact to the protocol is negligible because the fee collected would be minuscule.

## Proof of Concept
In SubprotocolRegistry, if a subprotocol is registered with a `_fee` that is greater than 0 but less than 10 it will cause the `cidFee` calculation in CidNFT.add to result in a 0 fee.

```solidity
function add(...){
       if (subprotocolFee != 0) {
            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000; //@audit subprotocolFee should always be set above 9 or the cidFee will be 0 due to rounding down on the div by 10_000.
            SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, cidFee);
            SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);
            }
}
```

Here's a foundry test for CidNFT.t.sol which shows that the CID fee is 0 when `subFee = 9`.

```
    function testLowFeeNotPaid() public { //@audit added by auditor
        // fee is set stupidly low, the result is that the cid-fee is not paid. Almost the same as effectively setting the fee to 0.
        uint96 subFee = 9;
        vm.startPrank(user1);
        subprotocolRegistry.register(true, true, true, address(sub1), "SubWithFee", subFee);
        vm.stopPrank();

        // mint fee
        note.mint(user2, 1000 * 1e18);

        // record balances
        uint256 balFeeWalletBefore = note.balanceOf(feeWallet);
        uint256 balRegistryOwnerBefore = note.balanceOf(address(user1));

        (uint256 tokenId, uint256 subId, uint256 key) = prepareAddOne(user2, user2);
        vm.startPrank(user2);
        note.approve(address(cidNFT), type(uint256).max);
        // add event
        vm.expectEmit(true, true, true, true);
        emit OrderedDataAdded(tokenId, "SubWithFee", key, subId);
        cidNFT.add(tokenId, "SubWithFee", key, subId, CidNFT.AssociationType.ORDERED);
        vm.stopPrank();
        // confirm data
        assertEq(cidNFT.getOrderedData(tokenId, "SubWithFee", key), subId);

        // check that the feeWallet balance has not increased
        assertEq(note.balanceOf(feeWallet),balFeeWalletBefore);
        console.log("feeWallet balance before add:",balFeeWalletBefore);
        console.log("feeWallet balance after add:",note.balanceOf(feeWallet));
        console.log("Owner of sub balance before add:", balRegistryOwnerBefore);
        console.log("Owner of sub balance after add:", note.balanceOf(address(user1)));
    }
```

```solidity
//Test results
Running 1 test for src/test/CidNFT.t.sol:CidNFTTest
[PASS] testLowFeeNotPaid() (gas: 337947)
Logs:
  feeWallet balance before add: 300000000000000000000
  feeWallet balance after add: 300000000000000000000
  Owner of sub balance before add: 9700000000000000000000
  Owner of sub balance after add: 9700000000000000000009

Test result: ok. 1 passed; 0 failed; finished in 2.04ms
```

## Tools Used
Manual analysis.

## Recommended Mitigation Steps
In `SubprotocolRegistry.register` Require that `_fee` is either 0 or greater than 9. 