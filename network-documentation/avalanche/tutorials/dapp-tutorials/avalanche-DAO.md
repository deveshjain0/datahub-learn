# How to Create a DAO with Avalanche

# Introduction

In this tutorial, we'll go over how to create a DAO and how to build the Solidity smart contract code that will be deployed on the Avalanche blockchain. We'll also create a user interface that works with smart contacts.

## What is a Decentralized Autonomous Organization (DAO)?

[Avalanche](https://support.avax.network/en/articles/4587123-what-is-a-decentralized-autonomous-organization-dao) defines that, a Decentralized Autonomous Organization (DAO) cooperates according to a transparent set of rules encoded on a blockchain, eliminating the need for a centralized, administrative, and potentially malicious entity. The reason some entity would want to create a DAO is to fundraise, manage financial operations, and/or decentralize governance to the community.

[](https://support.avax.network/en/articles/4587123-what-is-a-decentralized-autonomous-organization-dao)


## How do DAOs work?

[Ethereum](https://ethereum.org/en/dao/) defines that, the backbone of a DAO is its smart contract. The contract defines the rules of the organisation and holds the group's treasury. Once the contract is live on Ethereum, no one can change the rules except by a vote. If anyone tries to do something that's not covered by the rules and logic in the code, it will fail. And because the treasury is defined by the smart contract too, that means no one can spend the money without the group's approval either. This means that DAOs don't need a central authority. Instead, the group makes decisions collectively and payments are authorised automatically when votes pass.

This is possible because smart contracts are tamper-proof once they go live on Ethereum. You can't just edit the code (the DAO rules) without people noticing, because everything is public.

# Prerequisites

- Basic familiarity with [How do I set up MetaMask on Avalanche?](https://support.avax.network/en/articles/4626956-how-do-i-set-up-metamask-on-avalanche)
- Basic familiarity with [Avalanche's architecture](https://docs.avax.network/learn/platform-overview) and smart contracts.
- Basic familiarity with [ReactJS](https://reactjs.org/).
 
# Requirements

- [Remix - Ethereum IDE](https://remix.ethereum.org/).
- Metamask extension added to the browser, which must only be obtained from the official [Metamask website](https://metamask.io). Metamask should not be downloaded from an unofficial website.

# Let's start building our DAO

## Step 1: Creating a new .sol file on REMIX

On REMIX we click on the new file icon and put some name, in my case my file name is `MYDAO.sol`.

![](/.gitbook/assets/AVAX-MYDAO-SOL.png)

and we will add the basic lines of code:

The first line states that the source code is licenced under the **GPL version 3.0**. In a situation where publishing the source code is the default, machine-readable licencing specifiers are critical. **pragma** The source code is written for **Solidity** 0.7.0 or a newer version of the language up to but not including version 0.9.0. **contract MyDAO{..}** gives the name of our contract as well as a new block of code.

## Step 2: Defining our DAO functions

The DAO's contract typically has four main functions:
- Deposit governance tokens
- Withdraw the tokens
- Create a proposal
- Vote

We use AVAX our governance token. FUJI contract address: 0xA048B6a5c1be4b81d99C3Fd993c98783adC2eF70 and we need import IERC20 template from [openzeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sols).


## Step 3: Defining the proposal variables

For the proposal format we defined a group with custom properties, the properties for our proposal are:
- `Author`- which is an address from the account that create a proposal.
- `Id`- that will help us to identify a proposal.
- `Name`- of the proposal.
- `Creation date`- is a feature that allows us to specify a time limit for voting.
- `Voting options`- in this case we will keep it simple(Yes / NO).
- `Number of Votes for Yes and Votes for No`- this will allow us set a status for the proposal when number of votes for any option be greater than fifty percent.
- `Status for the Proposal`- this options will be Accepted, Rejected and Pending.

For the voting options and the proposal status we will use an enums types.

Enums are useful for creating custom types with a finite number of `constant values`. [see more about enums](https://docs.soliditylang.org/en/v0.8.7/types.html#enums)

```solidity
enum VotingOptions { Yes, No }
enum Status { Accepted, Rejected, Pending }
```

We can use a struct type for the remaining proposal properties.
Structs allow us to define a custom group of properties. [see more about structs](https://docs.soliditylang.org/en/v0.8.7/types.html#structs)

```solidity
    struct Proposal {
        uint256 id;
        address author;
        string name;
        uint256 createdAt;
        uint256 votesForYes;
        uint256 votesForNo;
        Status status;
    }
```

Until this step, our DAO contract looks like this:

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol';

contract MyDAO {
    
    enum VotingOptions { Yes, No }
    enum Status { Accepted, Rejected, Pending }
    struct Proposal {
        uint256 id;
        address author;
        string name;
        uint256 createdAt;
        uint256 votesForYes;
        uint256 votesForNo;
        Status status;
    }
    
}
```

Now we need to store all of the proposals that have been created for our DAO, make sure that no one votes twice, set a voting period for the proposals, and set a minimum number of governance tokens to create a new proposal. We can take the number of governance tokens that have been deposited like shares for an investor and give their vote a proportional weight.

```solidity
// store all proposals
mapping(uint => Proposal) public proposals;
// who already votes for who and to avoid vote twice
mapping(address => mapping(uint => bool)) public votes;
// one share for governance tokens
mapping(address => uint256) public shares;
uint public totalShares;
// the IERC20 allow us to use avax like our governance token.
IERC20 public token;
// the user need minimum 25 AVAX to create a proposal.
uint constant CREATE_PROPOSAL_MIN_SHARE = 25 * 10 ** 18;
uint constant VOTING_PERIOD = 7 days;
uint public nextProposalId;
```

## Step 4: The Deposit and Withdraw function for the DAO

We already have all of the variables we need to create, save, and vote on a proposal in our DAO,now we just need our user to deposit his AVAX tokens to prevent the same user from voting on the same proposal with the same number of tokens. To interact with AVAX as our token the governance we need to initialize the token address in the constructor.

```solidity
constructor() {
    token = IERC20(0xA048B6a5c1be4b81d99C3Fd993c98783adC2eF70); // AVAX address
}
```

For the deposit function.

```solidity
function deposit(uint _amount) external {
    shares[msg.sender] += _amount;
    totalShares += _amount;
    token.transferFrom(msg.sender, address(this), _amount);
}
```

When the voting time is over, we must allow our users to withdraw their tokens.

```solidity
function withdraw(uint _amount) external {
    require(shares[msg.sender] >= _amount, 'Not enough shares');
    shares[msg.sender] -= _amount;
    totalShares -= _amount;
    token.transfer(msg.sender, _amount);
}
```

Until now, our smart contract has looked like this:

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol';

contract MyDAO {
    
    enum VotingOptions { Yes, No }
    enum Status { Accepted, Rejected, Pending }
    struct Proposal {
        uint256 id;
        address author;
        string name;
        uint256 createdAt;
        uint256 votesForYes;
        uint256 votesForNo;
        Status status;
    }

    // store all proposals
    mapping(uint => Proposal) public proposals;
    // who already votes for who and to avoid vote twice
    mapping(address => mapping(uint => bool)) public votes;
    // one share for governance tokens
    mapping(address => uint256) public shares;
    uint public totalShares;
    // the IERC20 allow us to use avax like our governance token.
    IERC20 public token;
    // the user need minimum 25 AVAX to create a proposal.
    uint constant CREATE_PROPOSAL_MIN_SHARE = 25 * 10 ** 18;
    uint constant VOTING_PERIOD = 7 days;
    uint public nextProposalId;
    
    constructor() {
        token = IERC20(0xA048B6a5c1be4b81d99C3Fd993c98783adC2eF70); // AVAX address
    }
    
    function deposit(uint _amount) external {
        shares[msg.sender] += _amount;
        totalShares += _amount;
        token.transferFrom(msg.sender, address(this), _amount);
    }
    
    function withdraw(uint _amount) external {
        require(shares[msg.sender] >= _amount, 'Not enough shares');
        shares[msg.sender] -= _amount;
        totalShares -= _amount;
        token.transfer(msg.sender, _amount);
    }
}
```

## Step 5: Create a Proposal and Voting Features

We'll add a condition to our create Proposal function that states the user can't create a new proposal unless he has at least 25 AVAX tokens.

```solidity
function createProposal(string memory name) external {
    // validate the user has enough shares to create a proposal
    require(shares[msg.sender] >= CREATE_PROPOSAL_MIN_SHARE, 'Not enough shares to create a proposal');
    
    proposals[nextProposalId] = Proposal(
        nextProposalId,
        msg.sender,
        name,
        block.timestamp,
        0,
        0,
        Status.Pending
    );
    nextProposalId++;
}
```

We'll need the proposal's id and the vote option for the Voting function, and we'll validate that the user hasn't voted yet and that the vote period is still open.
Also, if the proposal receives more than half of the votes in one option, the proposal status must be changed to Accepted or Rejected.

```solidity
function vote(uint _proposalId, VotingOptions _vote) external {
    Proposal storage proposal = proposals[_proposalId];
    require(votes[msg.sender][_proposalId] == false, 'already voted');
    require(block.timestamp <= proposal.createdAt + VOTING_PERIOD, 'Voting period is over');
    votes[msg.sender][_proposalId] = true;
    if(_vote == VotingOptions.Yes) {
        proposal.votesForYes += shares[msg.sender];
        if(proposal.votesForYes * 100 / totalShares > 50) {
            proposal.status = Status.Accepted;
        }
    } else {
        proposal.votesForNo += shares[msg.sender];
        if(proposal.votesForNo * 100 / totalShares > 50) {
            proposal.status = Status.Rejected;
        }
    }
}
```

Finally our DAO contract looks like this.

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol';

contract MyDAO {
    
    enum VotingOptions { Yes, No }
    enum Status { Accepted, Rejected, Pending }
    struct Proposal {
        uint256 id;
        address author;
        string name;
        uint256 createdAt;
        uint256 votesForYes;
        uint256 votesForNo;
        Status status;
    }

    // store all proposals
    mapping(uint => Proposal) public proposals;
    // who already votes for who and to avoid vote twice
    mapping(address => mapping(uint => bool)) public votes;
    // one share for governance tokens
    mapping(address => uint256) public shares;
    uint public totalShares;
    // the IERC20 allow us to use avax like our governance token.
    IERC20 public token;
    // the user need minimum 25 AVAX to create a proposal.
    uint constant CREATE_PROPOSAL_MIN_SHARE = 25 * 10 ** 18;
    uint constant VOTING_PERIOD = 7 days;
    uint public nextProposalId;
    
    constructor() {
        token = IERC20(0xA048B6a5c1be4b81d99C3Fd993c98783adC2eF70); //AVAX address
    }
    
    function deposit(uint _amount) external {
        shares[msg.sender] += _amount;
        totalShares += _amount;
        token.transferFrom(msg.sender, address(this), _amount);
    }
    
    function withdraw(uint _amount) external {
        require(shares[msg.sender] >= _amount, 'Not enough shares');
        shares[msg.sender] -= _amount;
        totalShares -= _amount;
        token.transfer(msg.sender, _amount);
    }
    
    function createProposal(string memory name) external {
        // validate the user has enough shares to create a proposal
        require(shares[msg.sender] >= CREATE_PROPOSAL_MIN_SHARE, 'Not enough shares to create a proposal');
        
        proposals[nextProposalId] = Proposal(
            nextProposalId,
            msg.sender,
            name,
            block.timestamp,
            0,
            0,
            Status.Pending
        );
        nextProposalId++;
    }
    
    function vote(uint _proposalId, VotingOptions _vote) external {
        Proposal storage proposal = proposals[_proposalId];
        require(votes[msg.sender][_proposalId] == false, 'already voted');
        require(block.timestamp <= proposal.createdAt + VOTING_PERIOD, 'Voting period is over');
        votes[msg.sender][_proposalId] = true;
        if(_vote == VotingOptions.Yes) {
            proposal.votesForYes += shares[msg.sender];
            if(proposal.votesForYes * 100 / totalShares > 50) {
                proposal.status = Status.Accepted;
            }
        } else {
            proposal.votesForNo += shares[msg.sender];
            if(proposal.votesForNo * 100 / totalShares > 50) {
                proposal.status = Status.Rejected;
            }
        }
    }
}
```

## Step 6: Deploy our DAO contract on FUJI

Now we need to compile our contract. I'm using the 0.8.0 version of the compiler, and clicking on the `Compile` button.
In the environment section, we select `Injected Web3`, and in the account, we select our FUJI network in the metamask plugin. Make sure your account has the required avax for the deployment and the minimum for creating a proposal.
[Here you can find the Faucet](https://faucet.avax-test.network/).
Click on the `Deploy` button and `confirm` the transaction in REMIX and when the Metamask window appers, click on the `Estimated Gas fee` and change the `Gas price` from 25 to 225 (GWEI) and then click `confirm`.

If the contract is deployed successfully on FUJI we can see the succes transaction on the REMIX inspector.

![](/.gitbook/assets/create-avax-DAO.gif)

# Conclusion

Now you know about creating a DAO and how to build the Solidity smart contract code that will be deployed on the Avalanche blockchain.

If you had any difficulties following this tutorial or simply want to discuss Polygon tech with us you can [join our community today](https://community.figment.io/) or [Join our discord channel](https://discord.com/channels/741351331222126663/741354026670751764)!

# About the author
- [Devendra Yadav](https://community.figment.io/u/dev.koold)
- [Devesh Jain](https://community.figment.io/u/deveshjain08)

# References
- https://support.avax.network/en/articles/4587123-what-is-a-decentralized-autonomous-organization-dao
- https://ethereum.org/en/dao/
- https://support.avax.network/en/articles/4626956-how-do-i-set-up-metamask-on-avalanche
- https://docs.avax.network/learn/platform-overview





























































