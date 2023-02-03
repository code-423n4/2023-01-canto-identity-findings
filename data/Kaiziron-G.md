## [G-01] Use assembly to write storage values

Example :
```
    function updateOwner(address newOwner) public {
        owner = newOwner;
    }

    function assemblyUpdateOwner(address newOwner) public {
        assembly {
            sstore(owner.slot, newOwner)
        }
    }
```
*There is 1 instance of this issue*
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L127

---
## [G-02] Mark storage variables as `immutable` if they never change after contract initialization.

State variables can be declared as constant or immutable. In both cases, the variables cannot be modified after the contract has been constructed. For constant variables, the value has to be fixed at compile-time, while for immutable, it can still be assigned at construction time.

The compiler does not reserve a storage slot for these variables, and every occurrence is inlined by the respective value.

Compared to regular state variables, the gas costs of constant and immutable variables are much lower. For a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. This allows for local optimizations. Immutable variables are evaluated once at construction time and their value is copied to all the places in the code where they are accessed. For these values, 32 bytes are reserved, even if they would fit in fewer bytes. Due to this, constant values can sometimes be cheaper than immutable values.


*There is 1 instance of this issue*
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L37

---
## [G-03] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. Overflow/underflow can be checked in assembly to ensure safety.

Example :
```
    //addition in Solidity
    function addTest(uint256 a, uint256 b) public pure {
        uint256 c = a + b;
    }

    //addition in assembly
    function addAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := add(a, b)

            if lt(c, a) {
                mstore(0x00, "overflow")
                revert(0x00, 0x20)
            }
        }
    }
```

*There are 5 instances of this issue*
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L193
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L225
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L279
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L280

---
## [G-04] ++i/i++ should be unchecked{++I}/unchecked{I++} when it is not possible for them to overflow

Solidity version 0.8+ checks for overflow/underflow by default, if overflow/underflow isn’t possible, gas can be saved by using an unchecked block.

*There are 2 instances of this issue*
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L148
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L150

---
## [G-05] Setting the constructor to payable 

Making the constructor to payable can eliminate the check for `msg.value`, saving 13 gas on deployment

*There are 3 instances of this issue*
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L65
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L36
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L119
