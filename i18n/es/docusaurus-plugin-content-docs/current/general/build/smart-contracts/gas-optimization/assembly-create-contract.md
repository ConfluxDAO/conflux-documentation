---
displayed_sidebar: generalSidebar
---

# Using Assembly for Multiple Contract Deployments

When deploying multiple contracts in a single transaction, using assembly can lead to gas savings. While Solidity's standard contract creation syntax is convenient, implementing the same functionality with assembly can be more gas-efficient by reusing memory space.

Solidity treats contract creation similar to external calls, expanding memory for each deployment. By using assembly, we can bypass these memory expansion costs while maintaining the same functionality.

### Gas Comparison Example

Here's a comparative example showing both approaches:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Standard Solidity approach
contract SolidityDeployer {
    function deployContracts() external returns (Target, Target) {
        Target target1 = new Target();
        Target target2 = new Target();
        return (target1, target2);
    }
}

// Optimized assembly approach
contract AssemblyDeployer {
    function deployContracts() external returns (Target, Target) {
        bytes memory creationCode = type(Target).creationCode;
        address target1;
        address target2;
        
        assembly {
            // Deploy first contract
            target1 := create(0, add(creationCode, 0x20), mload(creationCode))
            
            // Deploy second contract using same memory space
            target2 := create(0, add(creationCode, 0x20), mload(creationCode))
            
            // Verify deployments succeeded
            if iszero(and(target1, target2)) {
                revert(0, 0)
            }
            
            // Store results in scratch space
            mstore(0x00, target1)
            mstore(0x20, target2)
            return(0x00, 0x40)
        }
    }
}

contract Target {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}
```

### Gas Analysis

| Implementation | Gas Cost | Gas Saved |
| -------------- | -------- | --------- |
| Solidity Deploy | 261,032  | -         |
| Assembly Deploy | 260,210  | 822       |

### Understanding the Assembly Implementation

The assembly deployment implementation consists of several key components:

1. **Creation Code Preparation**:
```solidity
bytes memory creationCode = type(Target).creationCode;
```

2. **Contract Creation**:
```solidity
target1 := create(0, add(creationCode, 0x20), mload(creationCode))
```

3. **Memory Management**:
- Reuses the same memory space for both deployments
- Uses scratch space (0x00-0x40) for storing return values
- Avoids unnecessary memory expansion

**Key Points:**

- Assembly deployments are more gas-efficient than Solidity's `new` keyword
- Gas savings come from reusing memory space and avoiding expansion
- Multiple contracts can be deployed while maintaining proper error handling

### Best Practices for Implementation

1. Use assembly deployments when creating multiple similar contracts
2. Maintain clear documentation of memory layout and operations
3. Always include deployment success verification
4. Consider the trade-off between gas efficiency and code complexity

### Security Considerations

While using assembly for contract deployment is safe when implemented correctly, keep in mind:

- Assembly code bypasses Solidity's safety checks
- Careful testing is required to ensure proper contract initialization
- Verify deployment success with explicit checks
- Document memory layout and operations thoroughly

**Note**: The gas savings shown are approximate and may vary depending on the Solidity version, optimization settings, and contract complexity.