# For more modern and easily readable code update imports

Description:
This update makes the code more clean and easily readable by the auditors and average people who want to review the code before using it. In order the code to be more modern and modular  **only import what you need**.

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L4
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L4-L7
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L4-L7

 **Here is a example on how this code can be modified**
From:

    import "solmate/tokens/ERC721.sol";
    import "solmate/tokens/ERC20.sol";
    import "solmate/utils/SafeTransferLib.sol";
    import "./CidSubprotocolNFT.sol";

To:

    import {ERC20} from "solmate/tokens/ERC20.sol";
    import {ERC721} from "solmate/tokens/ERC721.sol";
    import {CidSubprotocolNFT} from "./CidSubprotocolNFT.sol";
    import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
