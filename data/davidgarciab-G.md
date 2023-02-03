# Transfer funds after prechecks to revert quickly and save gas.

There is a transfer of funds and afterwards, the input variables are checked to assess if the transaction reverts, and after that a check for protocol name duplication.

Move the `safeTransferFrom` operation after the protocol name check (Line 90) to avoid extra gas after the transfer operation in case of a revert.

*There are 2 instances of this issue*

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L87-L90


# Move first revert checks that does not access storage but input variables

There are several checks with access to storage and we could save gas in case of revert checking first the light access one

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L183