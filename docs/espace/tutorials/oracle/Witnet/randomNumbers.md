---
sidebar_position: 1
title: Get Random Numbers
description: Learn how to Use Witnet Oracle on Conflux eSpace to Get Real-time Price Data
keywords:
  - Hardhat
  - Smart Contracts
  - Oracle
displayed_sidebar: eSpaceSidebar
---

# Step-by-Step Tutorial: Using Witnet Oracle on Conflux eSpace to Fetch Random Numbers

## Prerequisites

- Node.js and npm installed
- Hardhat installed (`npm install --save-dev hardhat`)
- Git installed

## 1. Initialize a Hardhat Project

```bash
mkdir witnet-conflux-project
cd witnet-conflux-project
npx hardhat
```

Follow the prompts to create a basic sample project.

## 2. Install Dependencies

```bash
npm install @nomiclabs/hardhat-ethers ethers
npm install @nomiclabs/hardhat-waffle ethereum-waffle chai
npm install @witnet/proxy @witnet/request @witnet/truffle-examples
```

## 3. Create a Solidity Contract to Fetch Random Numbers

Create a new file `contracts/RandomNumber.sol` and add the following code:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@witnet/proxy/contracts/UsingWitnet.sol";
import "@witnet/request/contracts/WitnetRequest.sol";

contract RandomNumber is UsingWitnet {
    WitnetRequest public witnetRequest;
    uint256 public randomNumber;

    event RandomNumberRequested(uint256 id);
    event RandomNumberReceived(uint256 randomNumber);

    constructor(WitnetRequest _witnetRequest) UsingWitnet(_witnetRequest.witnet()) {
        witnetRequest = _witnetRequest;
    }

    function requestRandomNumber() public payable {
        uint256 id = witnetPostRequest(witnetRequest);
        emit RandomNumberRequested(id);
    }

    function completeRandomNumberRequest() public witnetRequestAccepted {
        uint256 id = witnetReadRequestResult(msg.sender);
        randomNumber = id;
        emit RandomNumberReceived(randomNumber);
    }
}
```

## 4. Create a Deployment Script

Create a new file `scripts/deploy.js` and add the following code:

```javascript
async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying contracts with the account:", deployer.address);

  const balance = await deployer.getBalance();
  console.log("Account balance:", balance.toString());

  const WitnetRequest = await ethers.getContractFactory("WitnetRequest");
  const witnetRequest = await WitnetRequest.deploy();
  await witnetRequest.deployed();
  console.log("WitnetRequest deployed to:", witnetRequest.address);

  const RandomNumber = await ethers.getContractFactory("RandomNumber");
  const randomNumber = await RandomNumber.deploy(witnetRequest.address);
  await randomNumber.deployed();
  console.log("RandomNumber deployed to:", randomNumber.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

## 5. Configure Hardhat

Update `hardhat.config.js` to include the Conflux eSpace network configuration:

```javascript
require("@nomiclabs/hardhat-waffle");

module.exports = {
  solidity: "0.8.0",
  networks: {
    conflux: {
      url: "https://evm.confluxrpc.com",
      accounts: ["<YOUR_PRIVATE_KEY>"],
    },
  },
};
```

## 6. Compile and Deploy

Compile the smart contracts:

```bash
npx hardhat compile
```

Deploy the smart contracts:

```bash
npx hardhat run scripts/deploy.js --network conflux
```

## 7. Interact with the Contract

You can now interact with the deployed contract to request and fetch random numbers.
