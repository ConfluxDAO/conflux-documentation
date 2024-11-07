---
displayed_sidebar: generalSidebar
---

# Using assembly for 96-byte memory operations

When optimizing gas usage in smart contracts, using assembly for memory operations with data of 96 bytes or less can lead to significant gas savings. This optimization leverages Solidity's memory layout and scratch space to perform operations more efficiently.

### Understanding Solidity's Memory Layout

Solidity's memory layout consists of several important regions:

1. **Scratch Space** (0x00-0x40):
   - First 64 bytes of memory
   - Available for temporary operations
   - Guaranteed not to be overwritten unexpectedly

2. **Free Memory Pointer** (0x40-0x60):
   - 32 bytes used by Solidity to track the next available memory location
   - Updated when new memory allocations occur

3. **Zero Slot** (0x60-0x80):
   - 32 bytes where uninitialized dynamic memory data points to
   - Always contains zeros for uninitialized dynamic arrays and strings

### Gas Comparison Example

Here's a comparative example showing both approaches for event logging:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ExpensiveLogger {
    event BlockData(uint256 blockTimestamp, uint256 blockNumber, uint256 blockGasLimit);
    
    function returnBlockData() external {
        emit BlockData(block.timestamp, block.number, block.gaslimit);
    }
}

contract OptimizedLogger {
    event BlockData(uint256 blockTimestamp, uint256 blockNumber, uint256 blockGasLimit);
    
    function returnBlockData() external {
        assembly {
            mstore(0x00, timestamp())
            mstore(0x20, number())
            mstore(0x40, gaslimit())
            log1(0x00,
                0x60,
                0x9ae98f1999f57fc58c1850d34a78f15d31bee81788521909bea49d7f53ed270b
            )
        }
    }
}
```

### Gas Analysis

| Implementation     | Gas Cost | Gas Saved |
| ----------------- | --------- | --------- |
| Standard Logger   | 26,145    | -         |
| Assembly Logger   | 22,790    | 3,355     |

### Example: Optimized Hashing

Here's another example showing optimization for hashing struct data:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ExpensiveHasher {
    bytes32 public hash;
    
    struct Values {
        uint256 a;
        uint256 b;
        uint256 c;
    }
    Values values;

    function setOnchainHash(Values calldata _values) external {
        hash = keccak256(abi.encode(_values));
        values = _values;
    }
}

contract OptimizedHasher {
    bytes32 public hash;
    
    struct Values {
        uint256 a;
        uint256 b;
        uint256 c;
    }
    Values values;

    function setOnchainHash(Values calldata _values) external {
        assembly {
            let fmp := mload(0x40)
            calldatacopy(0x00, 0x04, 0x60)
            sstore(hash.slot, keccak256(0x00, 0x60))
            mstore(0x40, fmp)
        }
        values = _values;
    }
}
```

### Best Practices for Implementation

1. **Memory Pointer Management**:
   - Cache the free memory pointer when needed
   - Restore it before returning to Solidity code
   - Only required when mixing assembly with Solidity operations

2. **Use Cases**:
   - Event logging with up to 96 bytes of data
   - Hashing operations on small data sets
   - Memory operations that don't require expansion

3. **Limitations**:
   - Only suitable for operations with 96 bytes or less
   - Must handle memory pointer correctly when mixing with Solidity code

### Security Considerations

When implementing these optimizations:

- Test thoroughly to ensure data integrity
- Document assembly code clearly
- Be careful with memory pointer management
- Verify that optimizations don't introduce bugs

### Key Points to Remember

1. Scratch space (0x00-0x40) is safe for temporary operations
2. Free memory pointer must be managed when mixing with Solidity
3. Zero slot (0x60) must remain zero for Solidity's assumptions
4. These optimizations work best for operations ≤ 96 bytes

**Note**: Gas savings may vary depending on the specific implementation and Solidity version used.
