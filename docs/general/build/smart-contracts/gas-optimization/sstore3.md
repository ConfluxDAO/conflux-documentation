---
displayed_sidebar: generalSidebar
---

# Optimizing Gas Usage with SSTORE3 in Ethereum Smart Contracts

The Ethereum Virtual Machine (EVM) provides various opcodes for storing data, among which SSTORE and its derivatives like SSTORE2 and SSTORE3 play critical roles. These allow for different strategies in how data is stored on the blockchain, impacting gas costs associated with contract operations. SSTORE3, in particular, offers an efficient way to manage gas costs when storing data by making the address at which data is stored independent of the data itself.

## Overview of SSTORE3

SSTORE3 builds on the concept introduced by SSTORE2 but with an important distinction: it decouples the data being stored from the address calculation. This separation allows for more predictable and efficient address generation and can be crucial for optimizing gas costs in smart contracts where data storage and retrieval operations are frequent.

### How SSTORE3 Works

1. **Data Storage**: Initially, the data is stored in the contract's storage using the traditional SSTORE opcode.
2. **Address Generation**: A constant `INIT_CODE` is used in conjunction with CREATE2. This code reads the previously stored data and deploys it as bytecode at a new address. The address is computed using a provided salt and the `INIT_CODE`, making it predictable and independent of the data itself.
3. **Data Retrieval**: The data can be accessed by computing the address using the same salt and then using the EXTCODECOPY opcode to copy the data from the bytecode of the address where it was stored.

### Advantages of SSTORE3

- **Gas Efficiency**: By separating the data from the address calculation, SSTORE3 allows for more efficient gas usage, especially when combined with strategies like variable packing.
- **Predictability**: Addresses can be precomputed off-chain, which can be particularly useful in dApps requiring frequent data reads and writes.

## Example Usage

Here’s a practical example of how SSTORE3 can be implemented in a Solidity smart contract to optimize gas costs:

```solidity
pragma solidity ^0.8.0;

contract SSTORE3Example {
    // Storage slot for the data
    bytes32 private storedData;

    // Event to emit the address of the deployed data
    event DataStored(address indexed dataAddress);

    function storeData(bytes32 data, bytes32 salt) public {
        // Step 1: Store the data in a storage variable
        storedData = data;

        // Step 2: Deploy a new contract with the data using CREATE2
        bytes memory bytecode = abi.encodePacked(
            type(DataStorage).creationCode,
            abi.encode(storedData)
        );
        address dataAddress;
        assembly {
            dataAddress := create2(0, add(bytecode, 32), mload(bytecode), salt)
        }

        emit DataStored(dataAddress);
    }

    function retrieveData(address dataAddress) public view returns (bytes32) {
        bytes32 data;
        assembly {
            extcodecopy(dataAddress, data, 0, 32)
        }
        return data;
    }
}

contract DataStorage {
    constructor(bytes32 data) {
        assembly {
            sstore(0, data)
        }
    }
}
```

## Recommendations for Gas Optimization

- **Use SSTORE3 for Rare Writes, Frequent Reads**: SSTORE3 is particularly advantageous in scenarios where write operations are infrequent, but read operations are frequent.
- **Combine with Variable Packing**: When designing your contract’s storage layout, consider packing multiple smaller variables into single slots to further reduce gas costs.

By utilizing SSTORE3 and thoughtful storage practices, developers can significantly reduce the gas costs associated with their smart contracts.

Learn more about storage optimization: [Solidity Storage Layout](https://docs.soliditylang.org/en/v0.8.25/internals/layout_in_storage.html)
