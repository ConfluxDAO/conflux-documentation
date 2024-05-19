---
title: Deleting Structs Containing Dynamic Datatypes
displayed_sidebar: generalSidebar
---

When working with smart contracts in Solidity, it's essential to understand how the `delete` keyword interacts with structs that contain dynamic data types. Deleting a struct that includes mappings or dynamic arrays does not automatically delete the dynamic data within those structures. 

In Solidity, the `delete` keyword can only remove one storage slot at a time. If the storage slot contains references to other storage slots, those references will not be deleted.

Here is an example to illustrate this behavior:

```solidity
contract NestedMapping {
    mapping(uint256 => DataStruct) public dataMapping;
    
    struct DataStruct {
        mapping(uint256 => uint256) nestedMapping;
    }
    
    function addData(uint256 index) external {
        dataMapping[index].nestedMapping[5] = 6;
    }
    
    function getData(uint256 index) external view returns (uint256) {
        return dataMapping[index].nestedMapping[5];
    }
    
    function deleteData(uint256 index) external {
        delete dataMapping[index];
    }
}
```

Consider the following transaction sequence:
1. Call `addData(1)`
2. Call `getData(1)`, which returns `6`
3. Call `deleteData(1)`
4. Call `getData(1)` again, which still returns `6`

This behavior occurs because the `delete` keyword only sets the `dataMapping` entry to its default value, but it does not remove the underlying nested mapping data. In Solidity, mappings are never "empty." Accessing an item in a deleted mapping will return the default value for the datatype, rather than reverting the transaction.

## Best Practices for Managing Nested Structures

To handle this scenario securely and correctly, consider the following strategies:

### Clear Nested Mappings Manually

Ensure that nested mappings are cleared manually before deleting the outer structure. Here's an updated version of the previous contract:

```solidity
contract ImprovedNestedMapping {
    mapping(uint256 => DataStruct) public dataMapping;

    struct DataStruct {
        mapping(uint256 => uint256) nestedMapping;
    }
    
    function addData(uint256 index) external {
        dataMapping[index].nestedMapping[5] = 6;
    }
    
    function getData(uint256 index) external view returns (uint256) {
        return dataMapping[index].nestedMapping[5];
    }
    
    function clearNestedMapping(uint256 index) internal {
        // Assuming we know the keys to clear, in a real scenario this might involve more logic
        delete dataMapping[index].nestedMapping[5];
    }
    
    function deleteData(uint256 index) external {
        clearNestedMapping(index);
        delete dataMapping[index];
    }
}
```

### Use Enumerable Mappings

When dealing with mappings, consider using libraries like `EnumerableMap` from OpenZeppelin, which provide utilities to manage and iterate over mappings, making it easier to clear nested data.

### Avoid Complex Nested Structures

If possible, avoid deeply nested structures or mappings. Flattening your data structures can simplify the logic and reduce the risk of unintentionally retaining data.

## Conclusion

Understanding how the `delete` keyword interacts with structs containing dynamic data types is crucial for writing secure and efficient Solidity contracts. Always ensure that nested data is cleared explicitly if necessary, and consider simplifying data structures to avoid these issues.

By following these best practices, you can help ensure that your smart contracts manage their state correctly and securely.
