Gas Optimizations/QA

1) `i++` can be moved into an unchecked block here: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L150
2) typo on this line: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L145. "for add to add" should be "to add".
3) This can be made a constant: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L149.
4) https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L193 `subprotocolFee - cidFee` can be unchecked
5) This can be unchecked: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L279 since it's impossible that it'll go below 0
6) This can be unchecked: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L280 since it's impossible that it'll go below 0