# How to Create Your Own DAO with Avalanche


## What is a Decentralized Autonomous Organization (DAO)?

A Decentralized Autonomous Organization (DAO) cooperates according to a transparent set of rules encoded on a blockchain, eliminating the need for a centralized, administrative, and potentially malicious entity. The reason some entity would want to create a DAO is to fundraise, manage financial operations, and/or decentralize governance to the community.

[](https://support.avax.network/en/articles/4587123-what-is-a-decentralized-autonomous-organization-dao)


## How do DAOs work?

The backbone of a DAO is its smart contract. The contract defines the rules of the organisation and holds the group's treasury. Once the contract is live on Ethereum, no one can change the rules except by a vote. If anyone tries to do something that's not covered by the rules and logic in the code, it will fail. And because the treasury is defined by the smart contract too that means no one can spend the money without the group's approval either. This means that DAOs don't need a central authority. Instead the group makes decisions collectively and payments are authorised automatically when votes pass.

This is possible because smart contracts are tamper-proof once they go live on Ethereum. You can't just edit the code (the DAOs rules) without people noticing because everything is public.

## Requirements

REMIX IDE
Metamask Wallet

## Let's start to build our DAO

### Step 1: Creating a new .sol file on REMIX



On REMIX we click the new file icon and put some name, in my case my file name is MyDAO.sol


and we add the basic lines of code:

The first line tells you that the source code is licensed under the GPL version 3.0. Machine-readable license specifiers are important in a setting where publishing the source code is the default. `pragma` Specifies that the source code is written for Solidity version 0.7.0 or a newer version of the language up to, but not including version 0.9.0. `contract MyDAO {...}` specifies the name and a new block of code for our contract.

### Step 2: Defining our DAO functions

Commonly the DAO's contract has four main functions:

Deposit governance tokens.
Withdraw the tokens.
Create a proposal.
Vote.
We use AVAX our governance token. FUJI contract address: 0xA048B6a5c1be4b81d99C3Fd993c98783adC2eF70 and we need import IERC20 template from [openzeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol)


### Step 3: Defining the proposal variables

For the proposal format we defined a group with custom properties, the properties for our proposal are:

Author which is an address from the account that create a proposal.
Id that will help us to identify a proposal.
Name of the proposal.
Creation date, that allow us to set a period of time for allow the voting.
Voting options, in this case we will keep it simple(Yes / NO).
Number of Votes for Yes and Votes for No this will allow us set an status for the proposal when number of votes for any option be greater than fifty percent.
Status for the Proposal this options will be Accepted, Rejected, Pending.
For the voting options and the proposal status we will use an enums types.

Enums can be used to create custom types with a finite set of 'constant values'.[see more about enums](https://docs.soliditylang.org/en/v0.8.7/types.html#enums)

```CPP
enum VotingOptions { Yes, No }
enum Status { Accepted, Rejected, Pending }
```

for the other proposal properties we can use an struct type.
Structs alow us to define a custom group of properties. [see more about structs](https://docs.soliditylang.org/en/v0.8.7/types.html#structs)


```CPP
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


Until this step our Dao contract looks like this:


```CPP
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


Now we need to store all the proposals created for our DAO, we need to be sure that someone does not vote more than once, also set a period of vote for the proposals and set a minimum number of governance tokens to create a new proposal, we can take the number of governance tokens are deposited like a shares for an shareholder and give a proportional weight to their vote.


```CPP
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



### Step 4: Deposit and Withdraw function for the DAO
We already have our necessary variables to create, save and vote a proposal in our DAO, now we need our user deposit his AVAX tokens to avoid that the same user can use the same amount of tokens for vote other option in the same proposal. To interact with AVAX as our token the governance we need to initialize the token address in the constructor.


```CPP
constructor() {
    token = IERC20(0xA048B6a5c1be4b81d99C3Fd993c98783adC2eF70); // AVAX address
}
```


For the deposit function.



```CPP
function deposit(uint _amount) external {
    shares[msg.sender] += _amount;
    totalShares += _amount;
    token.transferFrom(msg.sender, address(this), _amount);
}
```


And we need to allow our users to withdraw their tokens when the voting period is over.

```CPP

function withdraw(uint _amount) external {
    require(shares[msg.sender] >= _amount, 'Not enough shares');
    shares[msg.sender] -= _amount;
    totalShares -= _amount;
    token.transfer(msg.sender, _amount);
}
```

until this point our smart contract look like this:

```CPP
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


### Step 5: Create a Proposal and Vote functions


For our createProposal function we will add the condition that if the user does not have minimum 25 AVAX tokens He cannot create a new proposal.

```CPP
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


For the Vote function we need to receive the id for the proposal and the vote choice, we will validate that the user has not voted already and the vote period is currently open.
Also we validate if the proposal has more than fifty percent of votes in one option we need to change the proposal status to Accepted or Rejected.

```CPP
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










### Step 6: Deploy our DAO contract on FUJI
Now we need compile our contract, I'm using the 0.8.0 version compiler, and click on the Compile button.

In the environment section we choose the Injected Web3 option, in account we chose an account from our metamask plugin in the FUJI network, make sure that your account have the necessary avax for the deploy and the minimum for create a proposal.
[Here you can find the Faucet](https://faucet.avax-test.network/).
Click on the Deploy button and confirm the transaction in REMIX and Metamask and await for a few seconds.



































































