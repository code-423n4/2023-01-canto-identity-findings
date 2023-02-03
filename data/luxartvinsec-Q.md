# 1. Integer overflow in fee calculation

Link : https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L85

Summary: 

The code contains an integer overflow vulnerability when calculating the fee in the `register` function. The fee, `_fee`, is of type `uint96` which has a maximum value of (2^96 - 1) / 10^18 =~ 80 billion. This means if a user sets `_fee` to a value greater than 80 billion, the calculation of `_fee` will overflow and produce incorrect results.

Impact: 

Incorrect fee calculation can result in incorrect balance transfers and could result in loss of funds for the registrant or the `cidFeeWallet`.

Recommendation: 

To mitigate this issue, `_fee` should be of type `uint256` which has a maximum value of 2^256 - 1, which is much larger than the maximum value of `_fee`.

# 2. Unchecked input in `register` function leading to potential unsafe usage of SafeTransferLib

Link : https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L79

Summary: 

The `register` function of the SubprotocolRegistry contract uses the `SafeTransferLib` library to transfer the `REGISTER_FEE` from the user's account to the `cidFeeWallet`. However, the function does not validate the inputs `_nftAddress` and `_fee`, which could result in unintended usage of the `SafeTransferLib`.

Impact: 

An attacker can exploit this vulnerability by providing a malicious NFT address, potentially causing the funds to be transferred to an unexpected address, leading to a loss of funds.

Recommendation: 

To mitigate this vulnerability, it is recommended to validate the inputs `_nftAddress` and `_fee` before calling the `safeTransferFrom` function. Additionally, a check should be added to ensure that `_nftAddress` adheres to the `CidSubprotocolNFT` interface.

Example:

```
function register(
        bool _ordered,
        bool _primary,
        bool _active,
        address _nftAddress,
        string calldata _name,
        uint96 _fee
    ) external {
    require(_nftAddress != address(0), "NFT address cannot be 0");
    require(_fee >= 0, "Fee must be non-negative");
    SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);
    ...
```

# 3. Old solidity version

All contracts using 0.8.0 Solidity version. Please upgrade contracts latest Solidity version(0.8.18).

# 4. Unchecked Reversion in Contract AddressRegistry

Link : https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L42

Summary: 

The contract `AddressRegistry` has a reversion that is not checked properly in the `register` function. The `revert` statement is called in case of an invalid NFT ownership, but the reversion is not caught and handled by the calling function. This could lead to unexpected behavior and contract failures.

Impact: 

The unchecked reversion could cause problems in the contract execution and lead to contract failure.

Recommendation: 

The `register` function should either have a try-catch block to handle the reversion or it should return a Boolean value indicating the success or failure of the registration operation. This way, the calling function can handle the reversion and make an informed decision on how to proceed.

Example:

```
function register(uint256 _cidNFTID) external returns (bool success) {
    if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender) {
        revert NFTNotOwnedByUser(_cidNFTID, msg.sender);
    }
    cidNFTs[msg.sender] = _cidNFTID;
    emit CIDNFTAdded(msg.sender, _cidNFTID);
    success = true;
}
```

# 5. Improper Authorization in NFT Minting Function

Link : https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L147

Summary: 

The `mint` function in the contract does not properly check for authorization before minting a new NFT, allowing any caller to mint an NFT. This can result in unauthorized NFTs being created and can have security implications.

Impact: 

Unauthorized individuals or malicious actors can exploit this vulnerability to mint NFTs and potentially manipulate the NFT ecosystem within the contract. This can also result in lost control over the NFTs and associated data within the contract, as well as financial losses for NFT holders.

Recommendation:

Implement proper authorization checks in the `mint` function to prevent unauthorized individuals from minting NFTs. This can be done by checking the caller's address against a whitelist of authorized addresses or by requiring the caller to pass a signature or other proof of authorization. An example of how to implement this is shown below:

```
function mint(bytes[] calldata _addList) external {
    // Add authorization check
    require(msg.sender == authorizedAddress, "Unauthorized caller");

    // Continue with rest of function as is
    _mint(msg.sender, ++numMinted); 
    //...
}
```

Where `authorizedAddress` is the address that is authorized to call the `mint` function and require checks the condition, and reverts the transaction if it is not met.
