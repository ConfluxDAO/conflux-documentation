---
sidebar_position: 1
title: Get Real-time Price Data
description: Learn how to Use Witnet Oracle on Conflux eSpace to Get Real-time Price Data
keywords:
  - Hardhat
  - Smart Contracts
  - Oracle
displayed_sidebar: eSpaceSidebar
---

# Using Witnet Oracle to Fetch CFX/USD Price on Conflux eSpace with Hardhat

This tutorial will guide you through setting up a project using Hardhat to fetch the CFX/USD price from the Witnet Oracle on Conflux eSpace.

## Prerequisites

- Node.js and npm installed
- Conflux Portal installed
- Basic understanding of Solidity and Hardhat

## Steps

### Step 1: Setting Up the Project

1. Create a new directory for your project and navigate into it:

   ```bash
   mkdir witnet-conflux && cd witnet-conflux
   ```

2. Initialize a new Hardhat project:

   ```bash
   npm init -y
   npm install --save-dev hardhat
   npx hardhat
   ```

3. Choose "Create an empty hardhat.config.js" when prompted.

### Step 2: Installing Dependencies

1. Install the required packages:

   ```bash
   npm install --save @witnet/request @witnet/witnet-solidity-bridge conflux-sdk ethers
   npm install --save-dev @nomiclabs/hardhat-ethers @nomiclabs/hardhat-waffle ethereum-waffle chai
   ```

2. Install Conflux-specific packages:
   ```bash
   npm install --save-dev hardhat-conflux
   ```

### Step 3: Configuring Hardhat

Update `hardhat.config.js` to include Conflux eSpace network configuration:

```javascript
require("@nomiclabs/hardhat-waffle");
require("hardhat-conflux");

module.exports = {
  solidity: "0.8.4",
  networks: {
    eSpace: {
      url: "https://evm.confluxrpc.com",
      accounts: ["YOUR_PRIVATE_KEY"],
    },
  },
};
```

Replace `YOUR_PRIVATE_KEY` with your actual Conflux eSpace account private key.

### Step 4: Writing the Smart Contract

Create a new file `contracts/PriceFeed.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@witnet/request/contracts/WitnetRequestBoard.sol";
import "@witnet/request/contracts/WitnetRequest.sol";
import "@witnet/request/contracts/UsingWitnet.sol";
import "@witnet/witnet-solidity-bridge/contracts/Request.sol";

contract PriceFeed is UsingWitnet {
    Request public request;
    uint256 public lastPrice;

    event PriceUpdated(uint256 price);

    constructor(WitnetRequestBoard _wrb, Request _request) UsingWitnet(_wrb) {
        request = _request;
    }

    function requestPrice() external payable {
        uint256 witnetReward = _witnetEstimateReward();
        _witnetPostRequest(request, witnetReward);
    }

    function completePriceRequest() external witnetRequestResolved(request.id()) {
        Witnet.Result memory result = _witnetReadResult(request.id());
        if (witnet.isOk(result)) {
            lastPrice = witnet.asUint64(result);
            emit PriceUpdated(lastPrice);
        } else {
            string memory errorMessage = witnet.asErrorMessage(result);
            revert(errorMessage);
        }
    }
}
```

### Step 5: Deploying the Contract

Create a new file `scripts/deploy.js`:

```javascript
async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying contracts with the account:", deployer.address);

  const WRB_ADDRESS = "0x...";
  const REQUEST_ADDRESS = "0x...";

  const PriceFeed = await ethers.getContractFactory("PriceFeed");
  const priceFeed = await PriceFeed.deploy(WRB_ADDRESS, REQUEST_ADDRESS);

  console.log("PriceFeed deployed to:", priceFeed.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Replace `WRB_ADDRESS` and `REQUEST_ADDRESS` with the actual addresses of the Witnet Request Board and your specific price request contract.

### Step 6: Running the Deployment Script

1. Compile the contracts:

   ```bash
   npx hardhat compile
   ```

2. Deploy the contract:
   ```bash
   npx hardhat run scripts/deploy.js --network eSpace
   ```

### Step 7: Interacting with the Contract

You can now interact with the deployed contract to request and fetch the CFX/USD price. Use the Conflux eSpace tools and your preferred method (such as a frontend application or directly via Hardhat scripts) to call the `requestPrice` and `completePriceRequest` functions.

### Conclusion

This tutorial covered the steps to set up a Hardhat project, write and deploy a Solidity contract using the Witnet Oracle on Conflux eSpace, and interact with the contract to fetch CFX/USD prices. For more advanced usage and customizations, refer to the official documentation of Hardhat, Conflux, and Witnet.
