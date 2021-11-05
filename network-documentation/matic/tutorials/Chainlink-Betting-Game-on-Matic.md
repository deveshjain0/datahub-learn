# Chainlink Betting Game on Polygon

# Introduction 
This is a blockchain based betting game where you can bet on the outcome of a dice roll with cryptocurrency and if you guess right, then you double your money. This game is powered by smart contacts that run on the Polygon blockchain. We're going to use the Chainlink protocol to implement randomness for our dice roll. 

The application will work when the user connects to the website with their crypto wallet (probably Metamask), they'll talk to a front-end application built in React.js and the application will talk directly to the blockchain. We'll create a smart contract, `BettingGame.sol`, that implements the betting game which will use the Chainlink protocol to interact with the Chainlink oracle smart contacts. Users will make a bet directly with our smart contracts with the funded application. If they guess the number right, they will win twice the amount of cryptocurrency that they bet.

# Prerequisites

- Basic familiarity with [Setting up MetaMask for Matic](https://medium.com/stakingbits/setting-up-metamask-for-polygon-matic-network-838058f6d844#:~:text=Setup%20MetaMask%20to%20connect%20to,Network%20Name%3A%20Polygon).
- Basic familiarity with [ReactJS](https://reactjs.org/).

# Requirements 

- [Node.js](https://nodejs.org/en/)
- [Chainlink Oracles](https://chain.link/)
- [Metamask](https://metamask.io/) 
- [Truffle](https://www.trufflesuite.com/truffle)
- [React.js](https://reactjs.org/)

# How to use Chainlink 

Chainlink is an external data provider for the Polygon blockchain, so it's an oracle service which means that it provides real-world data to smart contacts. In this tutorial, Chainlink is used to provide price feeds to smart contracts. We are also focusing on Chainlink's ability to provide randomness to smart contracts, which is an essential feature for the betting game.

# Betting game functionality

So basically the user makes a bet directly on our smart contract by calling the game function and what they do is they bet on a dice roll. They bet the lowest value or the highest value, which is going to be either **one to three** or **three to six**. They provide a random seed for that number and if they win, twice the amount of cryptocurrency that they bet is sent to them. If their bet is unsuccessful, then they lose the cryptocurrency.

We will go through:
- The Chainlink request & receive cycle
- Using the MATIC token
- How to use request & receive with Chainlink Oracles
- Consuming random numbers with Chainlink VRF in smart contracts

## 1. Request & Receive

The request and receive cycle describes how a smart contract requests data from an oracle and receives the response in a separate transaction. 
Chainlink VRF follows the Request & Receive Data cycle. To consume randomness, your contract should inherit from `VRFConsumerBase` and define two required functions

- `requestRandomness`, which makes the initial request for randomness.
- `fulfillRandomness`, which is the function that receives and provide verified randomness.

If the result of randomness is stored on-chain, any actor could see the value and predict the outcome. Instead, randomness must be requested from an oracle, which generates a number and a cryptographic proof then returns that result to the contract that requested it. This sequence is what's known as the [Request and Receive](https://docs.chain.link/docs/architecture-request-model/) cycle.

## 2. Using MATIC

In return for providing this service of generating a random number, Oracles need to be paid in MATIC tokens. This is paid by the contract that requests the randomness, and payment occurs during the request.

## 3. Interacting with Chainlink VRF

The Chainlink VRF (Verifiable Random Function) provides a fair and verifiable source of randomness for smart contracts. Developers of smart contracts may utilise Chainlink VRF as a tamper-proof random number generator (RNG) to build reliable smart contracts for any application which rely on unpredictable outcomes:

- Blockchain games and NFTs
- Random assignment of duties and resources (e.g. randomly assigning judges to cases)
- Choosing a representative sample for consensus mechanisms

When rolling the dice, it will accept an address variable to track which address is assigned to each house.
The contract will have the following functions:
- `rollDice`: This submits a randomness request to Chainlink VRF
- `fulfillRandomness`: The function that is used by the Oracle to send the result back to
- `house`: To see the assigned house of an address.

## Create a truffle project

Create a working directory: 

```text
mkdir workspace
```

Install Truffle:

```text
npm i -g truffle
```

Clone this [Git Repository](https://github.com/deveshjain0/chainlink_betting_game) and read the [Deploying and Debugging Smart Contracts on Polygon](https://learn.figment.io/tutorials/deploying-and-debugging-smart-contracts-on-polygon) tutorial to setup Truffle network config and learn about the deployment on the Polygon network.

```text
git clone https://github.com/deveshjain0/chainlink_betting_game
```

Go to the repository:

```text
cd chainlink_betting_game
```

Install the required depencencies:

```text
npm i
```
Now we will open `BettingGame.sol` in the `/contracts` directory.

## Importing VRFConsumerBase

Chainlink maintains a contract library that simplifies oracle data consumption. We use VRFConsumerBase for Chainlink VRF, which need to be imported and expanded from.

```solidity
pragma solidity 0.6.6;

import "https://raw.githubusercontent.com/smartcontractkit/chainlink/master/evm-contracts/src/v0.6/VRFConsumerBase.sol";
import "https://github.com/smartcontractkit/chainlink/blob/master/evm-contracts/src/v0.6/interfaces/AggregatorV3Interface.sol"; /* !UPDATE, import aggregator contract */

contract BettingGame is VRFConsumerBase {

}
```

## Contract variables

A number of items will be stored in the contract. To begin, it must store variables that tell the oracle about the request. Each Oracle job has its own Key Hash, which is used to identify which tasks it should perform. To use in the request, the contract will store the Key Hash that identifies Chainlink VRF, as well as the fee amount.

```solidity 
uint256 internal fee;
uint256 public randomResult;
```

Constructor inherits VRFConsumerBase and inside it initialize the `fee` as `0.1 LINK`, `admin` as `msg.sender` , `ethUsd` as `AggregatorV3Interface` value of [price feed](https://data.chain.link/ethereum/mainnet/crypto-usd/matic-usd) in USD.

```solidity
 constructor() VRFConsumerBase(VFRC_address, LINK_address) public {
    fee = 0.1 * 10 ** 18; // 0.1 LINK
    admin = msg.sender;
    ethUsd = AggregatorV3Interface(0xF9680D99D6C9589e2a93a78A04A279e509205945);
  }
```  

The contract will need to employ mappings to keep track of the addresses that roll the dice. Mappings are unique key => value pair data structures that act like hash tables.

``` solidity
uint256 public gameId;
uint256 public lastGameId;
address payable public admin;
mapping(uint256 => Game) public games;
```

## fulfillRandomness functionLink to this section

This is a special function defined within the `VRFConsumerBase` contract that ours extends from. It is the function that the coordinator sends the result back to, so we need to implement some functionality here to deal with the result.

It should:
- Transform the result to a number between 1 and 20 inclusively.
- Assign the transformed value to the address in the s_results mapping variable.
- Emit a DiceLanded event.

```solidity
  function getRandomNumber(uint256 userProvidedSeed) internal returns (bytes32 requestId) {
    require(LINK.balanceOf(address(this)) > fee, "Error, not enough LINK - fill contract with faucet");
    return requestRandomness(keyHash, fee, userProvidedSeed);
  }
```
## Random Number Consumer

Chainlink VRF follows the Request & Receive Data cycle. To consume randomness, your contract should inherit from VRFConsumerBase.

 1. requestRandomness, which makes the initial request for randomness.

   ```solidity
  function getRandomNumber(uint256 userProvidedSeed) internal returns (bytes32 requestId) {
    require(LINK.balanceOf(address(this)) > fee, "Error, not enough LINK - fill contract with faucet");
    return requestRandomness(keyHash, fee, userProvidedSeed);
  }
```
 2. fulfillRandomness, which is the function that receives and does something with verified randomness.

   ```solidity
  function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
    randomResult = randomness;

    //send final random value to the verdict();
    verdict(randomResult);
  }
```
 ## Send rewards to the winners. 
  
  If the user wins, they will get 2x the amount they wagered.
  
 ```solidity
    if((random>=half && games[i].bet==1) || (random<half && games[i].bet==0)){
        winAmount = games[i].amount*2;
        games[i].player.transfer(winAmount);
      }
      emit Result(games[i].id, games[i].bet, games[i].seed, games[i].amount, games[i].player, winAmount, random, block.timestamp);
    }
```

## Compile and migrate

Start the truffle console to run a local blockchain in your terminal at `http://127.0.0.1:9545/` with the command:

```text
truffle develop
```

This will display **Account addresses** along with their **Private Keys** and **Mnemonic** required for deploying the smart contracts.

In the truffle console, compile the smart contracts:

```text
truffle(develop)> compile

Compiling your contracts...
===========================
> Compiling .\contracts\BettingGame.sol

> Artifacts written to C:\Users\hp\BettingGame\build\contracts
> Compiled successfully using:
   - solc: 0.5.0+commit.1d4f565a.Emscripten.clang
```

Now, **migrate** the compiled smart contracts:

```text
truffle(develop)> migrate --network matic
```
BettingGame.sol smart contract is deployed succesfully. 

```text
 Deploying 'BettingGame'
   -----------------------
   > transaction hash:    0x232be40e9171c62f74585c52e15492a8a8653b8a65eb9f97f6e57ccdcb0eec66
   > Blocks: 0            Seconds: 0
   > contract address:    0xc7Eb239cA1e53093B645A50d70B4a895AAD94cb0
   > block number:        4
   > block timestamp:     1629372448
   > account:             0x2F3CeD6f849630301feC1dD613869E8cc3857665
   > balance:             99.990053476
   > gas used:            2460473 (0x258b39)
   > gas price:           2 gwei
   > value sent:          0 ETH
   > total cost:          0.004920946 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.004920946 ETH
   
Summary
=======
> Total deployments:   1
> Final cost:           0.004920946 ETH
- Saving migration to chain.
``` 

# Using the UI 

All the code is going to be inside the source directory (src).
Inside the Components directory is where the the application UI code is written, there are three main app components: `navbar.js` , `main.js` and `app.js`.

Open another terminal, and change into the project directory:

```text
cd chainlink_betting_game 
```
Then use the node package manager to run the start script contained in the chainlink_betting_game's package.json:

```text
npm run start 
```

This will run the web server on localhost.

Once the server has started, you can view the application in your browser. The Metamask extension will automatically pop up and let you connect to the app.

You have to make sure that you’re connected to the Polygon Mumbai testnet in Metamask, or otherwise add a custom RPC with the following parameters:

```text
Network Name: Mumbai Testnet
New RPC URL: https://rpc-mumbai.matic.today
Chain ID: 80001
Currency Symbol: MATIC
Block Explorer URL: https://explorer-mumbai.maticvigil.com/
```

You can see the account that you’re connected with here in the top right-hand corner. Betting game application on the top left-hand corner and here it is our little dice game. To play the game, well first we have got the max bet that’s the exact amount of MATIC cryptocurrency that we send to the smart contract. And the balance is the current wallet balance of your account which is connected to Metamask.

Go ahead and bet 1 MATIC to start playing.

![](/.gitbook/assets/chainlink-betting-game.gif)

# Conclusion


# About the author
- [Devendra Yadav](https://community.figment.io/u/dev.koold)
- [Devesh Jain](https://community.figment.io/u/deveshjain08)

# References
- https://learn.figment.io/tutorials/deploying-and-debugging-smart-contracts-on-polygon
- https://data.chain.link/ethereum/mainnet/crypto-usd/matic-usd
- https://blog.chain.link/random-number-generation-solidity/
- https://github.com/deveshjain0/chainlink_betting_game
