---
displayed_sidebar: generalSidebar
---

# Using Transfer Hooks for Token Transfers

In Solidity, using transfer hooks for token transfers can significantly optimize gas usage compared to traditional methods. This article explains how to leverage transfer hooks in various token standards to make your smart contracts more efficient.

Traditionally, the workflow for transferring tokens involves multiple steps and calls between contracts, which can be gas-intensive. By using transfer hooks, you can streamline this process, reducing the number of interactions and thereby saving on gas costs.

## Traditional Workflow vs. Transfer Hook

Let's compare the traditional workflow with the optimized approach using transfer hooks.

### Traditional Workflow

1. `msg.sender` approves contract A to accept token B.
2. `msg.sender` calls contract A to transfer tokens from `msg.sender` to A.
3. Contract A then calls token B to do the transfer.
4. Token B performs the transfer and calls `onTokenReceived()` in contract A.
5. Contract A returns a value from `onTokenReceived()` to token B.
6. Token B returns execution to contract A.

### Optimized Workflow with Transfer Hooks

1. `msg.sender` calls contract B to do a transfer.
2. Contract B calls the tokenReceived hook in contract A.

This method is more efficient as it reduces the number of external calls and interactions between contracts.

## Token Standards with Transfer Hooks

### ERC1155
- Includes a transfer hook.

### ERC721
- `safeTransfer` and `safeMint` have a transfer hook.

### ERC1363
- Includes `transferAndCall` functionality.

### ERC777
- Has a transfer hook but is deprecated. Use ERC1363 or ERC1155 for fungible tokens.

To pass arguments to contract A, you can use the data field and parse it in contract A.

## Demo Code

Below is an example demonstrating how to use transfer hooks with ERC1363 tokens.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

interface IERC1363 {
    function transferAndCall(address to, uint256 value, bytes calldata data) external returns (bool);
}

contract ContractA {
    event TokensReceived(address from, uint256 value, bytes data);

    function onTransferReceived(address operator, address from, uint256 value, bytes calldata data) external returns (bytes4) {
        emit TokensReceived(from, value, data);
        return this.onTransferReceived.selector;
    }
}

contract ContractB {
    IERC1363 public token;

    constructor(address tokenAddress) {
        token = IERC1363(tokenAddress);
    }

    function transferToA(address to, uint256 value, bytes calldata data) external {
        token.transferAndCall(to, value, data);
    }
}
```

### Explanation

1. `ContractA` implements the `onTransferReceived` function, which is called by the token contract when tokens are transferred.
2. `ContractB` interacts with the token contract using `transferAndCall`, which triggers the transfer and calls `onTransferReceived` in `ContractA`.

## Benefits of Using Transfer Hooks

- **Efficiency**: Reduces the number of external calls and interactions.
- **Gas Savings**: Fewer operations result in lower gas costs.
- **Simplicity**: Cleaner and more straightforward code.

By using transfer hooks, you can optimize your smart contract interactions, making them more efficient and cost-effective.

## Recommendations for Gas Optimization

- Use transfer hooks available in token standards like ERC1155, ERC1363, and ERC721.
- Avoid deprecated standards like ERC777 for new implementations.
- Leverage the data field to pass necessary arguments, minimizing the need for additional calls.

By adopting these practices, you can ensure your smart contracts are optimized for performance and cost.
