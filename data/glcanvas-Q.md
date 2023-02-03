## [N-1] Add possibility to iterates over subprotocols

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L25-L40

Now to fetch all possible subprotocols need to parse all blocks to fetch events, better to add additional array where will be storages all subprotocol names. 

----

## [N-2] Make add/remove logic more readable

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L159-L229
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L231-L288

Too complicated logic, split it by internal functions, like addOrdered, removeOrdered etc.