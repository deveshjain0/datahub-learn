# Build an NFT marketplace on Avalanche using HardHat

# Introduction 

In this guide you'll learn how to build, deploy, and test out a full stack NFT marketplace on Ethereum. We'll also look at how to deploy to Avalanche.

# Prerequisites

To be successful in this guide, you must have the following:

- Node.js installed on your machine
- Metamask wallet extension installed as a browser extension

# Requirements

- Node.js 
- HardHat is the framework we will be using for developing our smart contracts.
- MetaMask for interacting with the blockchain.

# About the project
The project that we will be building will be Metaverse Marketplace - a digital marketplace.

When a user puts an item for sale, the ownership of the item will be transferred from the creator to the marketplace. When a user purchases an item, the purchase price will be transferred from the buyer to the seller and the item will be transferred from the marketplace to the buyer. The marketplace owner will be able to set a listing fee. This fee will be taken from the seller and transferred to the contract owner upon completion of any sale, enabling the owner of the marketplace to earn recurring revenue from any sale transacted in the marketplace.

The marketplace logic will consist of two smart contracts:
- NFT Contract - This contract allows users to mint unique digital assets.
- Marketplace Contract - This contract allows users to put their digital assets for sale on an open market.

# Project setup

To get started, we'll create a new Next.js app. To do so, open your terminal. Create or change into a new empty directory and run the following command:

```
npx create-next-app digital-marketplace
```

Next, change into the new directory and install the dependencies:

```
cd digital-marketplace

npm install ethers hardhat @nomiclabs/hardhat-waffle \
ethereum-waffle chai @nomiclabs/hardhat-ethers \
web3modal @openzeppelin/contracts ipfs-http-client@50.1.2 \
axios
```

### Setting up Tailwind CSS

We'll be using [Tailwind CSS](https://tailwindcss.com/) for styling, we we will set that up in this step.

Tailwind is a utility-first CSS framework that makes it easy to add styling and create good looking websites without a lot of work.

Next, install the Tailwind dependencies:

```
npm install -D tailwindcss@latest postcss@latest autoprefixer@latest
```

Next, we will create the configuration files needed for Tailwind to work with Next.js (tailwind.config.js and postcss.config.js) by running the following command:

````
npx tailwindcss init -p
````

Finally, delete the code in styles/globals.css and update it with the following:
```
@tailwind base;
@tailwind components;
@tailwind utilities;
```
### Configuring Hardhat

Next, initialize a new Hardhat development environment from the root of your project:

```
npx hardhat

? What do you want to do? Create a sample project
? Hardhat project root: <Choose default path>
```

Now you should see the following files and folders created for you in your root directory:

- hardhat.config.js - The entirety of your Hardhat setup (i.e. your config, plugins, and custom tasks) is contained in this file.
- scripts - A folder containing a script named sample-script.js that will deploy your smart contract when executed
- test - A folder containing an example testing script
- contracts - A folder holding an example Solidity smart contract



























