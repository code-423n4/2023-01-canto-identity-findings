[NC-01] Subprotocol exist check should be in SubprotocolRegistry.sol

Currently there is no check if Subprotocol exist in SubprotocolRegistry.sol contract, instead it is made in the CidNFT.sol
[CidNFT.sol#L176](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L176)

Logically it should be presented in the getSubprotocol function of SubprotocolRegistry.sol, so in case of reuse of the SubprotocolRegistry.sol it would be already implemented
[SubprotocolRegistry.sol#L106](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L106)

