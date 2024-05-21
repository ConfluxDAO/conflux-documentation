---
displayed_sidebar: generalSidebar
---

# Metadata Optimization

In Solidity, the deployment cost of a smart contract can be significantly influenced by how the metadata is managed. This article explores various strategies for optimizing gas usage related to the compilation and deployment of smart contract metadata.

When compiling Solidity smart contracts, the compiler appends metadata at the end of the bytecode. This metadata includes information such as the IPFS hash of the source files and the version of the compiler used. While this metadata is crucial for contract verification, it increases the deployment gas cost since each byte of code deployed consumes gas.

**DemoCode**

Here's a breakdown of a simple smart contract and the impact of metadata on its deployment costs:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract SimpleStorage {
    uint256 storedData;

    constructor(uint256 initVal) payable {
        storedData = initVal;
    }
}


```

The contract literally does nothing. We can compile it to see the init code with solc --optimize-runs 1000 --bin C.sol. 

Compile with metadata
```bash
solc --optimize-runs 1000 --bin SimpleStorage.sol
```
We get the following output:
```
======= C.sol:Empty =======
Binary:
6080604052603e80600f5f395ff3fe60806040525f80fdfea26469706673582212203082dbb4f4db7e5d53b235f44d3e38f839dc82075e2cda9df05b88e6585bca8164736f6c63430008140033
```
Compile without metadata (--no-cbor-metadata)
```
solc --optimize-runs 1000 --bin --no-cbor-metadata SimpleStorage.sol
```
we get the following output:
```
======= C.sol:Empty =======
Binary:
6080604052600880600f5f395ff3fe60806040525f80fd
```

The bytecode output demonstrates a notable size reduction when the `--no-cbor-metadata` option is used, thereby reducing gas costs significantly at deployment.


By default, 51 bytes of metadata are appended. Removing these bytes can save approximately 10,200 gas during deployment. However, this must be balanced against the need for contract verification which relies on this metadata. It is not always ideal though as it can affect smart contract verification. Instead, developers can mine for code comments that make the IPFS hash that gets appendedhave more zeros in it.


**Recommendations for Developers**

🌟 Always evaluate the necessity of metadata for your use case. Consider removing it when verification is not a concern to optimize gas costs at deployment.

🌟 For contracts where verification is crucial, utilize techniques to optimize the IPFS hash, thus balancing verification needs with gas efficiency.
