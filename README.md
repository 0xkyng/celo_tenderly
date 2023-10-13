# How to Deploy, Debug, and Monitor a Solidity Smart Contract on Celo using Hardhat and Tenderly

## Abstract


Blockchain technology has revolutionized various industries, including the world of finance and payments. The Celo blockchain, known for its cost-effectiveness and scalability, has gained popularity as an alternative to Ethereum. 

In this article, we will provide a structured guide on how to deploy, debug, and monitor a Solidity smart contract on the Celo blockchain. We will use the example of a "Buy Me Coffee" smart contract to illustrate these processes. To accomplish this, we will employ the Hardhat development framework for contract deployment and utilize Tenderly, a powerful tool for Ethereum and Celo contract analysis, debugging and monitoring.

## Table of Content
<!-- TOC -->

- [How to Deploy, Debug, and Monitor a Solidity Smart Contract on Celo using Hardhat and Tenderly](#how-to-deploy-debug-and-monitor-a-solidity-smart-contract-on-celo-using-hardhat-and-tenderly)
  - [Abstract](#abstract)
  - [Table of Content](#table-of-content)
  - [Introduction](#introduction)
    - [Background on Celo](#background-on-celo)
  - [Requirements](#requirements)
  - [Setting Up the Project](#setting-up-the-project)
    - [Project Initialization](#project-initialization)
  - [Writing the Buy Me Coffee Smart Contract](#writing-the-buy-me-coffee-smart-contract)
  - [Configuring the Hardhat Project](#configuring-the-hardhat-project)
    - [Modifying the Configuration](#modifying-the-configuration)
  - [Compiling the Smart Contract](#compiling-the-smart-contract)
  - [Deploying the Buy Me Coffee Smart Contract to Celo Testnet](#deploying-the-buy-me-coffee-smart-contract-to-celo-testnet)
  - [Interacting with the Deployed Buy Me Coffee Contract](#interacting-with-the-deployed-buy-me-coffee-contract)
  - [Debugging with Hardhat and Tenderly](#debugging-with-hardhat-and-tenderly)
    - [Debugging with Hardhat](#debugging-with-hardhat)
    - [Debugging with Tenderly](#debugging-with-tenderly)
    - [Monitoring with Tenderly](#monitoring-with-tenderly)
  - [Conclusion](#conclusion)

<!-- /TOC -->


## Introduction 

### Background on Celo

Celo is a blockchain platform designed to make financial services and payments more accessible to people worldwide, particularly those in underserved regions. It utilizes a Proof of Stake (PoS) consensus mechanism to enhance scalability and reduce transaction costs. Similar to Ethereum, Celo supports smart contract development in the Solidity programming language.

## Requirements
Before we dive into the deployment, debugging, and monitoring of our "Buy Me Coffee" smart contract on the Celo blockchain, let's ensure we have the necessary tools and knowledge:

  1. Node.js: [Install Node.js](nodejs.org) 
  2. Package Manager: Choose either NPM or Yarn as your package manager. Both are widely used in the JavaScript ecosystem.

  3. Hardhat: Install Hardhat globally with NPM or Yarn:

 ```bash
    npm install -g hardhat
    # or
    yarn global add hardhat
 ```

 4. Tenderly Account: Sign up for a Tenderly account at [tenderly.co](tenderly.co).

 5. Solidity: Familiarize yourself with the Solidity programming language as it is essential for writing smart contracts.

 6. Celo Wallet: Install a Celo-compatible wallet like Valora or the Celo Extension Wallet for interacting with the Celo testnet and mainnet.

## Setting Up the Project
### Project Initialization

```bash
mkdir buy-me-coffee-contract
cd buy-me-coffee-contract
npx hardhat
```

Follow the prompts to set up your Hardhat project, selecting the option to "Create an empty Hardhat config file" if prompted.

## Writing the Buy Me Coffee Smart Contract

In this example, we will create a simple "Buy Me Coffee" smart contract. Users can send a Celo payment to the contract, and when a predefined payment threshold is reached, the contract owner is notified that they can enjoy a cup of coffee. Below is the Solidity code for our `BuyMeCoffee.sol` contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract BuyMeCoffee {
    address public owner;
    uint256 public coffeePrice;
    uint256 public coffeeCounter;

    event CoffeeBought(address indexed buyer, uint256 amount);

    constructor(uint256 _price) {
        owner = msg.sender;
        coffeePrice = _price;
        coffeeCounter = 0;
    }

    function buyCoffee() external payable {
        require(msg.value == coffeePrice, "Incorrect payment amount.");
        coffeeCounter++;
        emit CoffeeBought(msg.sender, msg.value);
    }

    function withdrawFunds() external {
        require(msg.sender == owner, "Only the owner can withdraw funds.");
        payable(owner).transfer(address(this).balance);
    }
}
```

Let's go through the code together;

Here, we declare a Solidity smart contract named `BuyMeCoffee`. This contract will allow users to buy coffee by sending a specific amount of cryptocurrency (in this case, Celo's native currency, cUSD) to the contract.

The contract includes three state variables:

- `owner`: This variable stores the address of the contract owner. The public keyword makes it publicly readable.

- `coffeePrice`: This variable represents the price of a cup of coffee in wei (the smallest unit of cryptocurrency). Users will need to send this amount when buying coffee.

- `coffeeCounter`: This variable keeps track of the total number of coffees bought by users.

There's also an event named `CoffeeBought`. Events are a way for smart contracts to communicate with external applications. This event will be emitted whenever a user buys a cup of coffee. It includes two parameters:

- `buyer`: The address of the buyer (indexed for efficient filtering).
- `amount`: The amount sent by the buyer in wei.

The constructor is a special function that is executed only once when the contract is deployed. It initializes the contract's state variables:

- `owner`: The `msg.sender` is set as the contract owner, which is the address that deploys the contract.

- `coffeePrice`: The price of a cup of coffee is set to the value passed as the `_price` parameter.

- `coffeeCounter`: The coffee counter is initialized to zero.

The `buyCoffee` function allows users to buy coffee by sending cryptocurrency to the contract. It has the `external` visibility, meaning it can be called from outside the contract.

The `payable` modifier indicates that this function can receive cryptocurrency.

The `require` statement checks if the amount sent (`msg.value`) is equal to the `coffeePrice`. If not, it reverts the transaction with the specified error message.

If the payment is correct, the `coffeeCounter` is incremented to record the purchase, and the `CoffeeBought` event is emitted to notify external applications about the purchase.

The `withdrawFunds` function allows the contract owner (the address that deployed the contract) to withdraw the accumulated funds from coffee purchases.

- It checks if the sender of the transaction (`msg.sender`) is the owner. If not, it reverts the transaction with an error message.

- If the sender is the owner, it uses `payable (owner).transfer(address(this).balance)` to transfer the contract's balance (accumulated funds) to the owner's address.

In summary, this "Buy Me Coffee" smart contract allows users to buy coffee by sending the correct amount of cryptocurrency. The contract owner can withdraw the accumulated funds. It also emits an event to notify external applications when a coffee is purchased. This example demonstrates how Solidity smart contracts can handle payments and events on the Celo blockchain.

## Configuring the Hardhat Project

### Modifying the Configuration

To configure the Hardhat project to work with the Celo network, open the `hardhat.config.js` file generated during project setup and modify it as follows:

```javascript
require('@nomiclabs/hardhat-waffle');
require('dotenv').config();

const CELO_RPC_URL = process.env.CELO_RPC_URL || 'https://forno.celo.org/';

module.exports = {
  defaultNetwork: 'hardhat',
  networks: {
    hardhat: {},
    celo: {
      url: CELO_RPC_URL,
      chainId: 42220,
      gas: 2000000,
      gasPrice: 1000000000, // 1 gwei
      accounts: {
        mnemonic: process.env.MNEMONIC || '',
      },
    },
  },
  solidity: {
    version: '0.8.0',
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
};
```

In this configuration:

- We include the Hardhat Waffle plugin for Celo integration.
- The default network is set to 'hardhat.'
- We define a 'celo' network with important parameters such as the Celo RPC URL, chain ID for the Celo Alfajores testnet, gas limit, gas price, and the account's mnemonic phrase. You can obtain testnet funds using the [Celo Faucet](https://celo.org/developers/faucet).

Make sure to set the `CELO_RPC_UR`L and `MNEMONIC `environment variables in your `.env` file.

## Compiling the Smart Contract

With the smart contract and project configuration in place, compile the contract using the following command:

```bash
npx hardhat compile
```

Hardhat will compile your contract, and you will find the compiled artifacts in the 'artifacts' directory.

## Deploying the Buy Me Coffee Smart Contract to Celo Testnet

Before deploying to the Celo testnet, create a deployment script. In your 'scripts' directory, create a new file named `deploy.js`:

```javascript
const { ethers, upgrades } = require('hardhat');

async function main() {
  const BuyMeCoffee = await ethers.getContractFactory('BuyMeCoffee');
  console.log('Deploying Buy Me Coffee contract...');
  const coffeeContract = await upgrades.deployProxy(BuyMeCoffee, [1000000000000000]); // Price in wei (1 cUSD)
  await coffeeContract.deployed();
  console.log('Buy Me Coffee contract deployed to:', coffeeContract.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });# How to Deploy, Debug, and Monitor a Solidity Smart Contract on Celo using Hardhat and Tenderly
```

This script deploys the `BuyMeCoffee` contract using the Hardhat Upgrades plugin. The coffee price is set to 0.000001 cUSD (1 cUSD = 1e18 wei).

Now, deploy the contract to the Celo testnet with the following command:

```bash
npx hardhat run scripts/deploy.js --network celo
```

After successful deployment, you will receive the contract address.

## Interacting with the Deployed Buy Me Coffee Contract

With the Buy Me Coffee contract deployed to the Celo testnet, let's interact with it using a JavaScript script. Create a new file in your project directory, name it `interact.js`, and add the following code:

```javascript
const { ethers } = require('hardhat');
const CELO_RPC_URL = process.env.CELO_RPC_URL || 'https://forno.celo.org/';

async function main() {
  const provider = new ethers.JsonRpcProvider(CELO_RPC_URL);
  const wallet = new ethers.Wallet.fromMnemonic(process.env.MNEMONIC);

  // Replace with the actual contract address deployed in the previous step
  const contractAddress = 'YOUR_CONTRACT_ADDRESS';
  const coffeeContract = await ethers.getContractAt('BuyMeCoffee', contractAddress);

  // Connect the wallet to the provider
  const connectedWallet = wallet.connect(provider);

  // Get the current coffee price
  const currentPrice = await coffeeContract.coffeePrice();
  console.log(`Current Coffee Price: ${currentPrice.toString()} wei`);

  // Buy coffee
  const coffeeTransaction = await coffeeContract.connect(connectedWallet).buyCoffee({
    value: currentPrice,
  });

  // Wait for the transaction to be mined
  await coffeeTransaction.wait();
  console.log('Coffee purchased!');

  // Check the coffee counter
  const coffeeCounter = await coffeeContract.coffeeCounter();
  console.log(`Total Coffees Bought: ${coffeeCounter.toString()}`);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

In this script:

- We import the necessary packages and set up a JSON RPC provider connected to the Celo testnet.

- We create a wallet from the mnemonic phrase to sign transactions.
Replace `'YOUR_CONTRACT_ADDRESS'` with the actual address of your deployed Buy Me Coffee contract.

- We retrieve the current coffee price and display it in wei.
- Coffee is purchased by calling the `buyCoffee` function with the appropriate value (matching the coffee price).
- We wait for the transaction to be mined and then display a confirmation message.
- Finally, we check the coffee counter to see how many coffees have been bought.

Execute this script with the following command:

```bash
node interact.js
```

This command will purchase a coffee and display the current coffee price and the total number of coffees bought.

## Debugging with Hardhat and Tenderly

Debugging smart contracts is a critical aspect of development. Hardhat provides built-in debugging capabilities, while Tenderly enhances the debugging experience. Let's explore how to use these tools to debug our Buy Me Coffee smart contract.

### Debugging with Hardhat

Hardhat includes a built-in Solidity debugger that allows you to step through your contract's code. To use it, you can add breakpoints in your contract and run the debugger with the following command:

```bash
npx hardhat node
```

This command starts a local Celo node with a built-in debugger. In your `interact.js` script, you can add breakpoints using `console.log` statements or by using `debugger`; statements in your contract code. Then, run your script in a separate terminal.

```bash
node interact.js
```

You'll see the debugger prompt in the terminal where you started the local node. You can enter commands like `cont` to continue execution, `step` to step through the code, and `print` `<variable>` to inspect variables. This can be extremely helpful for identifying and fixing issues in your contract.

### Debugging with Tenderly

Tenderly is a powerful tool for analyzing and debugging Ethereum and Celo smart contracts. It provides a user-friendly web interface to explore contract transactions, visualize contract state changes, and identify issues in your code.
Follow these steps to debug your Celo smart contract using Tenderly:

1. #### Install Tenderly CLI:

Install the Tenderly CLI to interact with your project from the command line:

```bash
npm install -g @tenderly/cli
```

2. #### Authenticate with Tenderly:

Authenticate your Tenderly CLI with your Tenderly account:

```bash
tenderly login
```

3. #### Create a Debug Script:

Create a debugging script, e.g., debug.js, to initiate the debugging process:

```javascript
const { Tenderly } = require('@tenderly/cli');

async function main() {
  const t = new Tenderly({
    project: 'YOUR_PROJECT_NAME',
    fork: 'https://alfajores-forno.celo-testnet.org', // Replace with Celo testnet fork URL
  });

  const trace = await t.createTrace({
    network: 'alfajores', // Replace with the network you're using
    tx: 'YOUR_TRANSACTION_HASH', // Replace with your transaction hash
  });

  console.log('Trace URL:', trace.url);
}

main().catch(console.error);
```

Make sure to replace `'YOUR_PROJECT_NAME'` with your actual Tenderly project name and provide the correct Celo testnet fork URL and transaction hash.

4. #### Run the Debugging Script:

Execute the debugging script:

```bash
node debug.js
```

This script will create a trace on Tenderly and provide a URL to access the debugging interface.

5. #### Debug with Tenderly UI:

Open the provided Trace URL in your browser. Tenderly's web interface allows you to step through the transaction's execution, inspect variable values, and identify any issues in your contract code.

### Monitoring with Tenderly

Tenderly also provides comprehensive contract monitoring capabilities. Here's how to set up contract monitoring with Tenderly:

1. Navigate to Your Tenderly Project:

Log in to your Tenderly account and navigate to your project containing the deployed smart contract.

2. Select Your Contract:

Choose the specific contract you want to monitor from the list of contracts in your project.

3. View Contract Dashboard:

You'll be presented with a dashboard displaying real-time information about your contract, including transaction activity, contract state changes, and gas usage.

4. Set Up Alerts:

You can configure alerts to be notified of specific contract events or anomalies. For example, you can set an alert to notify you when a large transaction occurs.

5. Utilize Analytics and Visualization Tools:

Tenderly provides various analytics and visualization tools that allow you to gain insights into your contract's performance, user interactions, and historical data.

By following these steps, you can effectively debug and monitor your Celo smart contracts using Tenderly's powerful features and user-friendly interface.

## Conclusion

In this article, we've provided a structured guide on deploying, debugging, and monitoring a Solidity smart contract on the Celo blockchain. We used the example of a "Buy Me Coffee" smart contract to illustrate each step of the process. Armed with this knowledge, you can confidently develop and maintain smart contracts on the Celo network, whether you're building financial applications, decentralized exchanges, or any other blockchain-based solution.
