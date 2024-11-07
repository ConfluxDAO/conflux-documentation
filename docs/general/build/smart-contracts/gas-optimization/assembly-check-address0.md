---
displayed_sidebar: generalSidebar
---

# Using assembly to check for address(0)

When optimizing gas usage in smart contracts, using assembly for address(0) checks can lead to significant gas savings. While Solidity's standard comparison operators are convenient, implementing the same functionality with assembly can be more gas-efficient.

Solidity's compiler adds additional overhead for standard address comparisons. By using assembly, we can perform these checks more efficiently while maintaining the same functionality.

### Gas Comparison Example

Here's a comparative example showing both approaches:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract StandardAddressCheck {
    function check(address _caller) public pure returns (bool) {
        require(_caller != address(0), "Zero address");
        return true;
    }
}

contract AssemblyAddressCheck {
    function check(address _caller) public pure returns (bool) {
        assembly {
            if iszero(_caller) {
                // Store offset to error message length
                mstore(0x00, 0x20)
                // Store length of error message
                mstore(0x20, 0x0c)
                // Store the error message
                mstore(0x40, 0x5a65726f20416464726573730000000000000000000000000000000000000000)
                // Revert with data (offset, size)
                revert(0x00, 0x60)
            }
        }
        return true;
    }
}
```

### Gas Analysis

| Implementation          | Gas Cost | Gas Saved |
| ---------------------- | -------- | --------- |
| Standard Address Check | 21,897   | -         |
| Assembly Address Check | 21,807   | 90        |

### Understanding the Assembly Implementation

The assembly implementation consists of several key components:

1. **Zero Check**:
```solidity
if iszero(_caller)
```
The `iszero` opcode checks if the address is zero (0x0)

2. **Memory Layout**:
- `0x00`: Stores the offset (0x20)
- `0x20`: Stores the error message length (0x0c = 12 bytes)
- `0x40`: Stores the actual error message "Zero Address" in hex

3. **Revert Operation**:
```solidity
revert(0x00, 0x60)
```
The first parameter (0x00) is the memory offset, and the second (0x60) is the size of the data to revert with.

### Key Benefits:

- More gas-efficient than standard Solidity address comparisons
- Reduces unnecessary compiler overhead
- Maintains the same functionality and error messages
- Particularly beneficial in frequently called functions

### Best Practices for Implementation

1. Use assembly address checks in gas-sensitive functions
2. Document the assembly code clearly
3. Consider the trade-off between code readability and gas optimization
4. Test thoroughly to ensure error messages are correctly displayed

### Security Considerations

While using assembly for address checks is safe when implemented correctly, keep in mind:

- Assembly code bypasses Solidity's safety checks
- Careful testing is required to ensure error messages are properly encoded
- Clear documentation is essential for maintainability
- Always verify the assembly implementation behaves identically to the standard check

**Note**: The gas savings shown are approximate and may vary depending on the Solidity version and optimization settings used.
