# Gas Report

## G-01: don't use SafeTransferFrom when transfering NFTs to own contract

You already know that your contract can handle these NFTs. Using SafeTransferFrom will only trigger the empty `onERC721Received()` function which will waste gas.

- https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L187

## G-02: don't use SafeERC20 on known ERC21 tokens

You already know the behaviour of the NOTE token. It's unnecwssary to use the SafeERC20 library when interacting with it.

- https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L192-L193
- https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L87

Those changes above lead to the following gas savings:
```
diff --git a/.gas-snapshot b/.gas-snapshot
index 999dba4..eb8b422 100644
--- a/.gas-snapshot
+++ b/.gas-snapshot
@@ -4,33 +4,33 @@ AddressRegistryTest:testRegisterNFTCallerNotOwner() (gas: 92924)
 AddressRegistryTest:testRemovePriorRegistration() (gas: 97653)
 AddressRegistryTest:testRemoveSecondTime() (gas: 99411)
 AddressRegistryTest:testRemoveWithoutRegister() (gas: 15684)
-CidNFTTest:testAddDuplicate() (gas: 277336)
+CidNFTTest:testAddDuplicate() (gas: 274639)
 CidNFTTest:testAddID0() (gas: 25857)
-CidNFTTest:testAddMultipleActiveTypeValues() (gas: 1079736)
+CidNFTTest:testAddMultipleActiveTypeValues() (gas: 1062876)
 CidNFTTest:testAddNonExistingSubprotocol() (gas: 19348)
-CidNFTTest:testAddRemoveActiveType() (gas: 245859)
-CidNFTTest:testAddRemoveByApprovedAccount() (gas: 227997)
-CidNFTTest:testAddRemoveByApprovedAllAccount() (gas: 229891)
-CidNFTTest:testAddRemoveByOwner() (gas: 207006)
-CidNFTTest:testAddRemoveByUnauthorizedAccount() (gas: 244213)
-CidNFTTest:testAddRemoveOrderedType() (gas: 206148)
-CidNFTTest:testAddRemovePrimaryType() (gas: 205184)
-CidNFTTest:testAddUnsupportedAssociationType() (gas: 2479112)
-CidNFTTest:testAddWithFee() (gas: 335150)
-CidNFTTest:testAddWithNotEnoughFee() (gas: 261628)
+CidNFTTest:testAddRemoveActiveType() (gas: 244510)
+CidNFTTest:testAddRemoveByApprovedAccount() (gas: 226648)
+CidNFTTest:testAddRemoveByApprovedAllAccount() (gas: 228542)
+CidNFTTest:testAddRemoveByOwner() (gas: 205657)
+CidNFTTest:testAddRemoveByUnauthorizedAccount() (gas: 242527)
+CidNFTTest:testAddRemoveOrderedType() (gas: 204799)
+CidNFTTest:testAddRemovePrimaryType() (gas: 203835)
+CidNFTTest:testAddUnsupportedAssociationType() (gas: 2443706)
+CidNFTTest:testAddWithFee() (gas: 333706)
+CidNFTTest:testAddWithNotEnoughFee() (gas: 259982)
 CidNFTTest:testCannotRemoveNonExistingEntry() (gas: 93432)
 CidNFTTest:testCannotRemoveWhenOrderedOrActiveNotSet() (gas: 108243)
-CidNFTTest:testMintWithMultiAddItems() (gas: 401911)
-CidNFTTest:testMintWithMultiAddItemsAndRevert() (gas: 264495)
-CidNFTTest:testMintWithSingleAddList() (gas: 204003)
+CidNFTTest:testMintWithMultiAddItems() (gas: 396853)
+CidNFTTest:testMintWithMultiAddItemsAndRevert() (gas: 262741)
+CidNFTTest:testMintWithSingleAddList() (gas: 202317)
 CidNFTTest:testMintWithoutAddList() (gas: 132322)
-CidNFTTest:testOverWritingPrimary() (gas: 279001)
-CidNFTTest:testOverwritingOrdered() (gas: 281662)
-CidNFTTest:testRemoveActiveValues() (gas: 1014427)
+CidNFTTest:testOverWritingPrimary() (gas: 275629)
+CidNFTTest:testOverwritingOrdered() (gas: 278290)
+CidNFTTest:testRemoveActiveValues() (gas: 1000939)
 CidNFTTest:testRemoveNonExistingSubprotocol() (gas: 90224)
 CidNFTTest:testTokenURI() (gas: 119858)
-IntegrationTest:testIntegrationCaseOne() (gas: 381096)
-IntegrationTest:testIntegrationCaseTwo() (gas: 395078)
+IntegrationTest:testIntegrationCaseOne() (gas: 377440)
+IntegrationTest:testIntegrationCaseTwo() (gas: 391421)
 SubprotocolRegistryTest:testCannotRegisterWithoutTypeSpecified() (gas: 807019)
 SubprotocolRegistryTest:testRegisterDifferentAssociation() (gas: 1789239)
 SubprotocolRegistryTest:testRegisterExistedProtocol() (gas: 863201)
```
