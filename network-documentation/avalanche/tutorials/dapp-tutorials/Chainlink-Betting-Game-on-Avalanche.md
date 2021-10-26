# Chainlink Betting Game on Matic

# Introduction 
This is a blockchain based betting game where you can bet on the outcome of a dice roll with cryptocurrency and if you guess right, then you double your money. This game is powered by ethereum smart contacts that run on the blockchain. And we're going to use the chainlink protocol to implement randomness for our dice roll. 

Here is the application will work when the user connects to their web browser with metamask, they'll talk to a front-end application built in react.js and the application will talk directly to the ethereum blockchain and on the blockchain, we'll create a smart contract that implements the betting game and that's going to use the chainlink protocol which of course talked to the chainlink smart contacts. So the user flow is here. Is that they make a bet to directly with our smart contacts with the funded application. If they guess the number right, they will win twice the amount of cryptocurrency that they bet. 

# Requirements 
- [Node.js](https://nodejs.org/en/)
- [Chainlink Oracles](https://chain.link/)
- [Metamask](https://metamask.io/) 

# Main game function

 Look at the chart, so basically the user makes a bet directly on our smart contract by calling the game function and what they do is they bet on a dice roll. And so they bet the lowest value or the highest value, which is going to be either `one to three` or `three to six`. They provide a random seed for that number and if they win twice the amount of cryptocurrency that they bet. And if not, then they lose the cryptocurrency. 

In this tutorial, we go through:
- The Chainlink request & receive cycle
- Using the LINK token
- How to use request & receive with Chainlink Oracles
- Consuming random numbers with Chainlink VRF in smart contracts

## 1. Request & Receive

The request and receive cycle describes how a smart contract requests data from an oracle and receives the response in a separate transaction. 
Chainlink VRF follows the Request & Receive Data cycle. To consume randomness, your contract should inherit from VRFConsumerBase and define two required functions

- `requestRandomness`, which makes the initial request for randomness.
- `fulfillRandomness`, which is the function that receives and does something with verified randomness.

If the result of randomness is stored on-chain, any actor could see the value and predict the outcome. Instead, randomness must be requested from an oracle, which generates a number and a cryptographic proof then returns that result to the contract that requested it. This sequence is what's known as the [Request and Receive](https://docs.chain.link/docs/architecture-request-model/) cycle.

## 2. Using LINK

In return for providing this service of generating a random number, Oracles need to be paid in Matic. This is paid by the contract that requests the randomness, and payment occurs during the request.

## 3. Interacting with Chainlink VRF

The Chainlink VRF (Verifiable Random Function) provides a fair and verifiable source of randomness for smart contracts. Developers of smart contracts may utilise Chainlink VRF as a tamper-proof random number generator (RNG) to build reliable smart contracts for any application which rely on unpredictable outcomes:

- Blockchain games and NFTs
- Random assignment of duties and resources (e.g. randomly assigning judges to cases)
- Choosing a representative sample for consensus mechanisms

When rolling the dice, it will accept an address variable to track which address is assigned to each house.
The contract will have the following functions:
- rollDice: This submits a randomness request to Chainlink VRF
- fulfillRandomness: The function that is used by the Oracle to send the result back to
- house: To see the assigned house of an address


### 3a. Importing VRFConsumerBase
Chainlink maintains a contract library that simplifies oracle data consumption. We use VRFConsumerBase for Chainlink VRF, which need to be imported and expanded from.

```cpp
pragma solidity 0.6.6;

import "https://raw.githubusercontent.com/smartcontractkit/chainlink/master/evm-contracts/src/v0.6/VRFConsumerBase.sol";
import "https://github.com/smartcontractkit/chainlink/blob/master/evm-contracts/src/v0.6/interfaces/AggregatorV3Interface.sol"; /* !UPDATE, import aggregator contract */

contract BettingGame is VRFConsumerBase {

}
```

### 3b. Contract variables

A number of items will be stored in the contract. To begin, it must store variables that tell the oracle about the request. Each Oracle job has its own Key Hash, which is used to identify which tasks it should perform. To use in the request, the contract will store the Key Hash that identifies Chainlink VRF, as well as the fee amount.

```cpp 

uint256 internal fee;
uint256 public randomResult;

```
These will be initialized in the constructor.

Constructor inherits VRFConsumerBase. 
```cpp
 constructor() VRFConsumerBase(VFRC_address, LINK_address) public {
    fee = 0.1 * 10 ** 18; // 0.1 LINK
    admin = msg.sender;
    ethUsd = AggregatorV3Interface(0xF9680D99D6C9589e2a93a78A04A279e509205945);
  }
  ```  

The contract will need to employ mappings to keep track of the addresses that roll the dice. Mappings are unique key => value pair data structures that act like hash tables.

```cpp 
uint256 public gameId;
uint256 public lastGameId;
address payable public admin;
mapping(uint256 => Game) public games;
```

### 3c. fulfillRandomness functionLink to this section

This is a special function defined within the VRFConsumerBase contract that ours extends from. It is the function that the coordinator sends the result back to, so we need to implement some functionality here to deal with the result.

It should:
- Transform the result to a number between 1 and 20 inclusively.
- Assign the transformed value to the address in the s_results mapping variable.
- Emit a DiceLanded event.

```cpp
  function getRandomNumber(uint256 userProvidedSeed) internal returns (bytes32 requestId) {
    require(LINK.balanceOf(address(this)) > fee, "Error, not enough LINK - fill contract with faucet");
    return requestRandomness(keyHash, fee, userProvidedSeed);
  }
```
### 3d. Random Number Consumer

Chainlink VRF follows the Request & Receive Data cycle. To consume randomness, your contract should inherit from VRFConsumerBase.

 1. requestRandomness, which makes the initial request for randomness.

   ```cpp
  function getRandomNumber(uint256 userProvidedSeed) internal returns (bytes32 requestId) {
    require(LINK.balanceOf(address(this)) > fee, "Error, not enough LINK - fill contract with faucet");
    return requestRandomness(keyHash, fee, userProvidedSeed);
  }
```
 2. fulfillRandomness, which is the function that receives and does something with verified randomness.

   ```cpp
  function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
    randomResult = randomness;

    //send final random value to the verdict();
    verdict(randomResult);
  }
  ```
 ### 3e. Send rewards to the winners. 
  
  If the user wins, they will get 2x the amount they wagered.
  
 ```cpp    
    if((random>=half && games[i].bet==1) || (random<half && games[i].bet==0)){
        winAmount = games[i].amount*2;
        games[i].player.transfer(winAmount);
      }
      emit Result(games[i].id, games[i].bet, games[i].seed, games[i].amount, games[i].player, winAmount, random, block.timestamp);
    }
  ```
  
  
# How to use chainlink 

Chainlink is an external data provider for the blockchain,so its an oracle service which means that it provides real-world data to smart contacts. 

In this chainlink is use to provide cryptocurrency prices to smart contracts like exchanges
 We focus on today is chainlink ability to provide randomness to smart contracts which is an essential feature for gaming there are some inherent 

# Node.js 
 After installation go to your terminal typing 

node -v 

# Metamask 

Google metamask web extension

# Clone github to terminal 

Open Terminal then type 
```
git clone “link”
```

After downloading. Then, type in terminal 
```
cd chainlink_betting_game 
```

To install all dependencies for the projects with `npm`

In terminal.          
```
npm  install 
```

Once all your dependencies install go ahead and open up a project text editor 


All the code is going to be inside the source directory (src).   Contact directory for the smart contracts that we’ll put on the ethereum blockchain 

So `Bettingame.sol`.  Is going to be the main smart contract that we’re going to use for this tutorial 

Components directory is for the application code written,this is the main app component like navbar.js and main.js and app.js 



Open file explorer application.   Copy the bettingGame.sol code and paste in the application. Then compile first , use 0.66.  Because we’re using a solidity version for this particular tutorial ,then compile it . 
Make sure everything works properly 
Next we want to deploy it and put it on the blockchain so we’re using the test network for this . 
We’re going to use the rinkeby test network, in order to do that 



In metamask. You have to make sure that you’re connected to the rinkeby test network. 

Open terminal. 
```
cd chainlink_betting_game 
```
Then Type 
```
npm run start 
```
This will run the web server

Once its done you can see the application loaded in your browser here . This should be automatically popup. 

You can see your account that you’re connected with here in the top right hand corner 
Betting game application on top left hand corner 
And here it is our little dice game and we can play the game ,well first we have got the max bet that’s the exact amount of ethereum cryptocurrency thar we send to the smart contract. And the balance is your current wallet balance of your account which is connected to the metamask 

Lets bet the 0.5 ethereum, then lets playing.
