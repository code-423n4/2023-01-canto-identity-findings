# Non-critical
## Use keyword `ether` instead of 10**18

In Solidity 1 ether equals to 10 ** 18. 
You can rewriting [REGISTER_FEE](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L17) constant like this:
uint256 public constant REGISTER_FEE = 100 ether;

## Check whether the fee wallet is safe

SubprotocolRegistry & CidNFT smart contracts have wallets that receive fees paid:
- [SubprotocolRegistry.sol#L23](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L23)
- [CidNFT.sol#L24](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L24)

The `cidFeeWallet` variables are public variables. Make sure that addresses that will be used are safe.

# Low
## Multiple users can register the same a CID NFT

Publicly known issue that user can transfer his CID NFT while it is still registered to his address.
But there is such a case when `user1` will mint a CID NFT, register and transfer his CID NFT to `user2`. `user2` will register the same CID NFT and transfer `user3` and so on. After that the AddressRegistry smart contract has multiple users that registered the same a CID NFT.
Consider adding a revert to the `register` function if the same CID NFT has already registered.

Test:

```
pragma solidity >=0.8.0;

import {DSTest} from "ds-test/test.sol";
import {Utilities} from "./utils/Utilities.sol";
import {console} from "./utils/Console.sol";
import {Vm} from "forge-std/Vm.sol";
import "../AddressRegistry.sol";
import "../SubprotocolRegistry.sol";
import "../CidSubprotocolNFT.sol";
import "./mock/MockERC20.sol";
import "./mock/SubprotocolNFT.sol";
import "../CidNFT.sol";

contract Tester is DSTest {
    Vm internal immutable vm = Vm(HEVM_ADDRESS);

    Utilities internal utils;
    address payable[] internal users;
    AddressRegistry internal addressRegistry;

    CidNFT cidNFT;

    SubprotocolRegistry subprotocolRegistry;
    MockToken token;

    address feeWallet;
    address user1;
    address user2;
    address user3;
    address user4;
    address user5;

    uint256 feeAmount;

    function setUp() public {
        utils = new Utilities();
        users = utils.createUsers(10);

        user1 = users[0];
        user2 = users[1];
        user3 = users[2];
        user4 = users[3];
        user5 = users[4];
        feeWallet = users[9];

        token = new MockToken();
        subprotocolRegistry = new SubprotocolRegistry(
            address(token),
            feeWallet
        );

        feeAmount = subprotocolRegistry.REGISTER_FEE();

        vm.prank(user1);
        token.approve(address(subprotocolRegistry), type(uint256).max);
        token.mint(user1, feeAmount * 100);

        cidNFT = new CidNFT(
            "MockCidNFT",
            "MCNFT",
            "base_uri/",
            users[0],
            address(token),
            address(subprotocolRegistry)
        );
        addressRegistry = new AddressRegistry(address(cidNFT));
    }

    function testMintOneCidNftAndRegisterEndlessTimesFromDifferentUsers()
        public
    {
        uint256 nftId = cidNFT.numMinted() + 1;

        // user mint NFT
        vm.startPrank(user1);
        bytes[] memory addList;
        cidNFT.mint(addList);
        assertEq(cidNFT.ownerOf(nftId), user1);

        // user register a CID NFT
        addressRegistry.register(nftId);
        uint256 cid = addressRegistry.getCID(user1);
        assertEq(cid, nftId);

        // transfer the CID NFT to the user2 & register
        cidNFT.transferFrom(user1, user2, nftId);
        assertEq(cidNFT.ownerOf(nftId), user2);
        vm.stopPrank();
        vm.startPrank(user2);
        addressRegistry.register(nftId);

        // transfer the CID NFT to the user3 & register
        cidNFT.transferFrom(user2, user3, nftId);
        assertEq(cidNFT.ownerOf(nftId), user3);
        vm.stopPrank();
        vm.startPrank(user3);
        addressRegistry.register(nftId);

        // transfer the CID NFT to the user4 & register

        cidNFT.transferFrom(user3, user4, nftId);
        assertEq(cidNFT.ownerOf(nftId), user4);
        vm.stopPrank();
        vm.startPrank(user4);
        addressRegistry.register(nftId);

        // transfer the CID NFT to the user5 & register
        cidNFT.transferFrom(user4, user5, nftId);
        assertEq(cidNFT.ownerOf(nftId), user5);
        vm.stopPrank();
        vm.startPrank(user5);
        addressRegistry.register(nftId);

        // One CID NFT has registered from the different addresses in AddressRegistry
        cid = addressRegistry.getCID(user1);
        assertEq(cid, nftId);
        cid = addressRegistry.getCID(user2);
        assertEq(cid, nftId);
        cid = addressRegistry.getCID(user3);
        assertEq(cid, nftId);
        cid = addressRegistry.getCID(user4);
        assertEq(cid, nftId);
        cid = addressRegistry.getCID(user5);
        assertEq(cid, nftId);

        vm.stopPrank();
    }
}
```

