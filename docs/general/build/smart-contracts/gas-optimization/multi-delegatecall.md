---
displayed_sidebar: generalSidebar
---

# Efficient Use of Multi-delegatecall

This tutorial explores how using multi-delegatecall to batch transactions can result in substantial gas savings. Multi-delegatecall allows developers to call multiple functions within a contract while preserving environment variables like `msg.sender` and `msg.value`, leading to more efficient smart contract execution.

In Solidity, multi-delegatecall helps to execute a sequence of function calls within the same context, preserving the `msg.sender` and `msg.value` across these calls. However, developers need to be cautious with the persistence of `msg.value` to avoid unintended issues.

**Demo Code**

Below is an example of a contract using multi-delegatecall for batching transactions, inspired by Uniswap's implementation.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MultiDelegatecall {
    function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {
        results = new bytes[](data.length);
        for (uint256 i = 0; i < data.length; i++) {
            (bool success, bytes memory result) = address(this).delegatecall(data[i]);
            if (!success) {
                if (result.length < 68) revert();
                assembly {
                    result := add(result, 0x04)
                }
                revert(abi.decode(result, (string)));
            }
            results[i] = result;
        }
    }
}
```

The `multicall` function demonstrates how to batch multiple function calls while preserving `msg.sender` and `msg.value`. This can lead to significant gas savings by reducing the overhead associated with multiple external calls.

**Recommendations for Gas Optimization**

🌟 Use multi-delegatecall to batch transactions, reducing the number of external calls and associated gas costs.

🌟 Be mindful of the persistence of `msg.value` across calls to avoid unintended issues. Ensure proper handling of `msg.value` to maintain contract security.

🌟 Test thoroughly to ensure that batched function calls behave as expected in all scenarios.

By implementing these practices, developers can optimize gas usage in their smart contracts while maintaining secure and efficient execution.
