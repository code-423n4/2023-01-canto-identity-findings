# For more modern and easily readable code update imports

Description:
This update makes the code more clean and easily readable by the auditors and average people who want to review the code before using it. In order the code to be more modern and modular  **only import what you need**.

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
