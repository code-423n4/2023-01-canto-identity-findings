## Use internal function instead of self delegate call

CidNFT.mint delegate call to self to allow adding subprotocol in the same call. Instead of using a delegate call, use a switch statement with abi.decode and internal function would be cheaper and safer.

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L154

## Use of string can be expensive

Consider using hash of the string, or the contract address when supplying contract arguments or emitting event. String, especially when long, can be expensive to process in EVM.

