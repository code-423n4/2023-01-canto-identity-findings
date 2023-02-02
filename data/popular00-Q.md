# QA
- [NC-1] `struct SubprotocolData` is defined in different ways in `CidNFT.sol` and `SubprotocolRegistry.sol`. `SubprotocolRegistry.SubprotocolData` is then used several times in `CidNFT.sol`. Recommend using different naming conventions to more clearly define data structures.

    `CidNFT.sol` 
    ```     
    /// @notice Data that is associated with a CID NFT -> subprotocol combination
    struct SubprotocolData {
        /// @notice Mapping for ordered type
        mapping(uint256 => uint256) ordered;
        /// @notice Value for primary type
        uint256 primary;
        /// @notice List for active type
        IndexedArray active;
    } 
    ```

    `SubprotocolRegistry.sol`
    ```
    /// @notice Data that is associated with a subprotocol.
    /// @dev Data types are chosen such that all data fits in one slot
    struct SubprotocolData {
        /// @notice Owner (registrant) of the subprotocol
        address owner;
        /// @notice Optional cost in NOTE to add an NFT
        /// @dev Maximum value is (2^96 - 1) / 10^18 =~ 80 billion. Zero for no fee
        uint96 fee;
        address nftAddress;
        bool ordered;
        bool primary;
        bool active;
    }
    ```
- [NC-2] `struct SubprotocolData` in `SubprotocolRegistry.sol` comments indicate all data fits in one storage slot, however the struct as written will be packed into two slots
- [NC-3] Code section labeling is inconsistent - some sections include the solmate section delimiter while others don't