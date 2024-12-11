---
displayed_sidebar: generalSidebar
---

# Using Bitwise Operations for Even/Odd Number Checks

When optimizing gas usage in smart contracts, using bitwise operations to check if a number is even or odd can lead to gas savings. While Solidity's modulo operator (`%`) is commonly used, implementing the same functionality with bitwise operations can be more gas-efficient.

### Gas Comparison Example

Here's a comparative example showing both approaches:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ModuloChecker {
    function isEven(uint256 number) external pure returns (bool) {
        return number % 2 == 0;
    }
    
    function isOdd(uint256 number) external pure returns (bool) {
        return number % 2 == 1;
    }
}

contract BitwiseChecker {
    function isEven(uint256 number) external pure returns (bool) {
        return number & 1 == 0;
    }
    
    function isOdd(uint256 number) external pure returns (bool) {
        return number & 1 == 1;
    }
}
```

### Gas Analysis

| Implementation | Gas Cost (isEven) | Gas Saved |
| -------------- | ----------------- | --------- |
| Modulo Check   | 351               | -         |
| Bitwise Check  | 330               | 21        |

### Understanding the Implementation

The bitwise implementation works because:

1. **Binary Number Structure**:
   - Even numbers end in 0 in binary
   - Odd numbers end in 1 in binary

2. **Bitwise AND Operation**:
   - `number & 1` masks all bits except the least significant bit
   - If the result is 0, the number is even
   - If the result is 1, the number is odd

### Technical Explanation

In binary:
- Even numbers: ...000, ...010, ...100, etc.
- Odd numbers:  ...001, ...011, ...101, etc.

When we perform `number & 1`:
- For even numbers: xxxxx0 & 000001 = 0
- For odd numbers:  xxxxx1 & 000001 = 1

### Best Practices for Implementation

1. Use bitwise operations for even/odd checks in gas-sensitive functions
2. Document the logic clearly for maintainability
3. Test thoroughly with various input numbers
4. Consider code readability vs. gas optimization trade-offs

### Security Considerations

While using bitwise operations is safe, keep in mind:

- The logic is less intuitive than modulo operations
- Clear documentation is important for code maintenance
- Test edge cases thoroughly

**Note**: The gas savings shown are approximate and may vary depending on the Solidity version and optimization settings used.