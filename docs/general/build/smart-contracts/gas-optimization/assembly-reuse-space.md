---
displayed_sidebar: generalSidebar
---

# Using assembly to optimize memory in external calls

When making multiple external calls in smart contracts, using assembly to manage memory can lead to significant gas savings. While Solidity automatically handles memory management, it doesn't reuse memory spaces efficiently between calls, leading to unnecessary memory expansion.

### Gas Comparison Example

Here's a comparative example showing both approaches:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}

contract SolidityCall {
    function makeExternalCalls(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);
        return res1 + res2;
    }
}

contract AssemblyCall {
    function makeExternalCalls(address calledAddress) external view returns(uint256) {
        assembly {
            // Verify contract exists at address
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // Store function selector for add(uint256,uint256)
            mstore(0x00, hex"771602f7")
            
            // First call parameters
            mstore(0x04, 0x01)  // First parameter (1)
            mstore(0x24, 0x02)  // Second parameter (2)
            
            // Make first call
            let success := staticcall(
                gas(),
                calledAddress,
                0x00,    // Input offset
                0x44,    // Input size (4 + 32 + 32)
                0x60,    // Output offset
                0x20     // Output size
            )
            
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // Reuse same memory space for second call
            mstore(0x04, 0x03)  // First parameter (3)
            mstore(0x24, 0x04)  // Second parameter (4)
            
            // Make second call
            success := staticcall(
                gas(),
                calledAddress,
                0x00,    // Reuse same input offset
                0x44,    // Same input size
                0x60,    // Reuse same output offset
                0x20     // Same output size
            )
            
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // Store final result
            mstore(0x60, add(res1, res2))
            return(0x60, 0x20)
        }
    }
}
```

### Gas Analysis

| Implementation     | Gas Cost | Gas Saved |
| ----------------- | --------- | --------- |
| Solidity Calls    | 7,262     | -         |
| Assembly Calls    | 5,281     | 1,981     |

### Understanding the Assembly Implementation

The assembly implementation optimizes memory usage through several key techniques:

1. **Memory Layout**:
   - `0x00`: Function selector (4 bytes)
   - `0x04-0x23`: First parameter (32 bytes)
   - `0x24-0x43`: Second parameter (32 bytes)
   - `0x60`: Output storage (32 bytes)

2. **Memory Reuse**:
   - The same memory space is reused for both calls
   - Output is stored in the same location (0x60)
   - No unnecessary memory expansion between calls

3. **Scratch Space Usage**:
   - Uses the first 96 bytes of memory (scratch space)
   - Efficient for function calls with small parameters

### Best Practices

1. **When to Use Assembly Memory Optimization**:
   - Multiple external calls in sequence
   - Function parameters fit within scratch space (<96 bytes)
   - High-frequency functions where gas optimization is crucial

2. **Memory Management**:
   - Update free memory pointer if using memory above 0x80
   - Clear the zero slot (0x60) before exiting if using dynamic memory
   - Carefully manage memory when dealing with dynamic data types

### Security Considerations

When implementing assembly memory optimizations:

- Test thoroughly to ensure proper memory management
- Be cautious with memory overlaps
- Document memory layout clearly
- Verify contract existence before calls
- Consider the trade-off between optimization and code readability

### Important Notes

1. For single external calls with large parameters (>64 bytes), assembly optimization might not provide significant benefits

2. Always update the free memory pointer when using memory above offset 0x80

3. Be careful not to overwrite the zero slot (0x60) when using undefined dynamic memory values

4. Consider explicitly defining dynamic memory values or resetting slots before exiting assembly blocks

**Note**: Gas savings may vary depending on the Solidity version, optimization settings, and specific implementation details.
