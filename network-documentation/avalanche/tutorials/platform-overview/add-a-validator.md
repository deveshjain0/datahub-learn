# Add a Validator

## Introduction

The [Primary Network](https://support.avax.network/en/articles/4135650-what-is-the-primary-network) is incorporated into the Avalanche platform and legitimize Avalanche’s [`built-in blockchains`](https://avalanche.gitbook.io/avalanche/learn/platform-overview). In this tutorial, we’ll create a [`subnet`](https://learn.figment.io/tutorials/create-a-subnet) on Avalanche and a node to the Primary Network. 


On Avalanche, the P-Chain regulates metadata.  This involves keeping track of which nodes belong to which subnets, which blockchains are active, and which subnets validate which blockchains. We’ll issue [`transactions`](http://support.avalabs.org/en/articles/4587384-what-is-a-transaction) to the P-Chain in order to add a validator. 

> Note: That there is no way to modify the parameters once you've issued the transaction to add a node as a validator. **You cannot amend the stake amount, node ID, or reward address, or withdraw your stake early.** Please double-check that the values in the API calls below are correct. If you’re not sure, browse the [Developer FAQ's](https://figment.io/contact/) or ask for help on [Discord](https://discord.gg/fszyM7K)

## Requirements

You've accomplished [`Run an Avalanche Node`](https://docs.avax.network/build/tutorials/nodes-and-staking/run-avalanche-node) and are acquainted with [`Avalanche's architecture`](https://docs.avax.network/learn/platform-overview#:~:text=Avalanche%20features%203%20built%2Din,secured%20by%20the%20Primary%20Network.) to assist us make API calls.

Make sure your node can receive and send TCP traffic on the staking port \(`9651` by default\) and that you initiated the node with the command line parameter `--public-ip=[YOUR NODE'S PUBLIC IP HERE]` to ensure your node is well-connected. If you fail to do either of these, your staking reward could be jeopardized.

## Create an Avalanche Wallet
 Make an [Avalanche Wallet] by clicking on this link `https://wallet.avax.network`
 ![Create an Avalanche Wallet](/.gitbook/assets/create-new-wallet.png)
 
## Home page of an Avalanche Wallet    
![Home page](/.gitbook/assets/wallet-ava-home.png)

## Add a validator with Avalanche Wallet

First, We've demonstrated how to add your node as a validator by using [Avalanche Wallet](https://wallet.avax.network).

Get your node’s ID by calling [`info.getNodeID`](https://docs.avax.network/build/tutorials/nodes-and-staking/add-a-validator):

![getNodeID postman](/.gitbook/assets/get-node-id-postman.PNG)

```cpp
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.getNodeID"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

The response would've your node’s ID:

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "nodeID": "NodeID-5mb46qkSBj81k9g9e4VFjGGSbaaSLFRzD"
    },
    "id": 1
}
```

Open [the Avalanche wallet](https://wallet.avax.network/), and go to the `Earn` tab.
![Web wallet earn tab](/.gitbook/assets/ava-earn-wallet.png)

Select this network option to enable Validator.
![wallet.ava-change-network-to-fuji](/.gitbook/assets/wallet.ava-change-network-to-fuji.png)

Select the Fuji option instead of Mainnet.
![select-fuji-network](/.gitbook/assets/select-fuji-network.png)

To  `Add Validator` you need to add coins in the Wallet.

For adding the coin in the Wallet, Click on this Link https://faucet.avax-test.network
![avax-request-fund](/.gitbook/assets/avax-request-fund.png)
Add your wallet Address, and verify Recaptcha, and hit on Request AVAX.

Now you have to convert your coin `X-Chain` to `P-Chain`, by clicking on the Cross Chain Tab.
![convert-x2p chain-ava-wallet](/.gitbook/assets/convert-x2p-chain-ava.png)
Put the Amount and hit `confirm`.
 

Complete the staking parameters. We've discussed in greater depth further down. After you've double-checked all of the staking parameters, click `Confirm` Make sure your staking term is at least two weeks long, the delegation fee rate is at least 2%, and you're staking at least 2,000 AVAX.

### [`Learn how to stake on Avalanche`](https://docs.avax.network/learn/platform-overview/staking) 
![`Earn validate`](/.gitbook/assets/earn-vadilator-ava-wallet.png)

This success notice should appear, along with an update to your balance.

![Your validation transaction is sent](/.gitbook/assets/x2p-conform-ava.png)

Calling [`platform.getPendingValidators`](https://docs.avax.network/build/tutorials/nodes-and-staking/add-a-validator) confirms that your transaction was successfully accepted.

![getPendingValidators postman](/.gitbook/assets/getPendingValidators-postman.png)

Go back to the `Earn` tab, and click `Estimated Rewards`.

![Earn, validate, delegate](/.gitbook/assets/earn-validate-delegate.png)

Once its start time has passed, you'll see the rewards your validator may have earned, as well as its start time, end time, and the percentage of its validation period that has elapsed.

![Estimated rewards](/.gitbook/assets/estimated-rewards.png)

That’s it!

## Add a validator with API calls

By making API calls to our node, we can also add a node to the validator set. To add a node to the Primary Network, we’ll call [`platform.addValidator`](https://docs.avax.network/build/tutorials/nodes-and-staking/add-a-validator#:~:text=Add%20a%20validator%20with%20Avalanche%20Wallet&text=Open%20the%20wallet%2C%20and%20go,Choose%20Add%20Validator%20.&text=Fill%20out%20the%20staking%20parameters,explained%20in%20more%20detail%20below.)

This method’s signature is:

```cpp
platform.addValidator(
    {
        nodeID: string,
        startTime: int,
        endTime: int,
        stakeAmount: int,
        rewardAddress: string,
        changeAddr: string, (optional)
        delegationFeeRate: float,
        username: string,
        password: string
    }
) -> {txID: string}
```

Let’s go through and examine these arguments.

`nodeID`

This is the node ID of the validator being added. To obtain your node’s ID, call [`info.getNodeID`](https://docs.avax.network/build/tutorials/nodes-and-staking/add-a-validator):

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "info.getNodeID",
    "params":{},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

The response has your node’s ID:

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "nodeID": "NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji"
    },
    "id": 1
}
```

`startTime` and `endTime`

When someone initiates a transaction to join the Primary Network, they must mention the time they want to enter. \(start validating\) and leave \(stop validating.\) The Primary Network can be validated for a minimum of 24 hours and a maximum of one year. After leaving the Primary Network, one can rejoin , however, the maximum _continuous_ period is one year. The Unix times when your validator will start and stop validating the Primary Network are `startTime` and `endTime` respectively. In relation to the time the transaction is issued,`startTime` must be in the future.

`stakeAmount`

Staking AVAX is required to validate the Primary Network. The quantity of AVAX staked is defined by this parameter.
`rewardAddress`

When a validator stops validating the Primary Network, they will receive a reward if they are sufficiently responsive and correct while they validated the Primary Network. These tokens are sent to `rewardAddress`. The original stake will be returned to a 'username' controlled address.

The stake of a validator is never slashed, regardless of their actions; they always get their stake back when they're finished verifying.

`changeAddr`

This address will receive any changes resulting from this transaction. You can leave this field blank; if you do, money will be sent to one of your user's addresses.

`delegationFeeRate`

Stake delegation is possible with Avalanche. When others delegate stake to this validator, this parameter is the percentage fee they charge. For example, if the `delegationFeeRate` is `1.2345` and someone delegates to this validator, then 1.2345 percent of the reward goes to the validator and the remainder goes to the delegator when the delegation period ends.

`username` and `password`

These are the username and password of the user who pays the transaction fee, provides the staked AVAX, and gets the staked AVAX returned.

Now it's time to issue the transaction. For the values of `startTime` and `endTime` we utilize the shell command `date` to compute the Unix time 10 minutes and 30 days in the future, respectively. \(Note: If you’re on a Mac, replace `$(date` with `$(gdate`. If you don’t have `gdate` installed, do `brew install coreutils`.\) In this example we stake 2,000 AVAX \(2 x 1012 nAVAX\).

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addValidator",
    "params": {
        "nodeID":"NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
        "startTime":'$(date --date="10 minutes" +%s)',
        "endTime":'$(date --date="30 days" +%s)',
        "stakeAmount":2000000000000,
        "rewardAddress":"P-avax1d4wfwrfgu4dkkyq7dlhx0lt69y2hjkjeejnhca",
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u",
        "delegationFeeRate":10,
        "username":"USERNAME",
        "password":"PASSWORD"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The transaction ID is included in the response, as well as the address where the change went to. 

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "txID": "6pb3mthunogehapzqmubmx6n38ii3lzytvdrxumovwkqftzls",
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u"
    },
    "id": 1
}
```

We can verify the status of the transaction by calling [`platform.getTxStatus`](https://docs.avax.network/build/avalanchego-apis/issuing-api-calls):

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getTxStatus",
    "params": {
        "txID":"6pb3mthunogehapzqmubmx6n38ii3lzytvdrxumovwkqftzls"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The status should be `Committed`, which indicates that the transaction was completed successfully. We can call [`platform.getPendingValidators`](https://docs.avax.network/build/tutorials/nodes-and-staking/add-a-validator) and see that the node is now in the pending validator set for the Primary Network:

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response should include the node we just added:

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "validators": [
            {
                "nodeID": "NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
                "startTime": "1584021450",
                "endtime": "1584121156",
                "stakeAmount": "2000000000000",
            }
        ] 
    },
    "id": 1
}
```

This node will commence validating the Primary Network when the time reaches `1584021450`. This node will stop validating the Primary Network when it reaches `1584121156`. The staked AVAX will be returned to an address controlled by `username` and any rewards will be sent to `rewardAddress` if there are any.

## Adding a Subnet Validator

### Issuing a Subnet Validator Transaction

Now let’s add the same node to a subnet. The following will make more sense if you’ve already done this [tutorial on creating a Subnet](https://docs.avax.network/build/tutorials/platform/create-a-subnet). Right now you can only add validators to subnets with API calls, not with Avalanche Wallet.

Suppose that the Subnet has ID `nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr`, threshold 2, and that `username` holds at least 2 control keys.

To add the validator, we’ll call API method [`platform.addSubnetValidator`](https://docs.avax.network/build/tutorials/nodes-and-staking/add-a-validator#:~:text=Add%20a%20validator%20with%20Avalanche%20Wallet&text=Open%20the%20wallet%2C%20and%20go,Choose%20Add%20Validator%20.&text=Fill%20out%20the%20staking%20parameters,explained%20in%20more%20detail%20below.). Its signature is:

```cpp
platform.addSubnetValidator(
    {
        nodeID: string,
        subnetID: string,
        startTime: int,
        endTime: int,
        weight: int,
        changeAddr: string, (optional)
        username: string,
        password: string
    }
) -> {txID: string}
```

Let’s examine the parameters:

`nodeID`

This is the node ID of the validator being added to the subnet. **This validator must validate the Primary Network for the entire duration that it validates this Subnet.**

`subnetID`

This is the ID of the subnet to which a validator will be added.

`startTime` and `endTime`

Similar to above, The validator will start and end validating the subnet at these Unix times. The validator must start validating the Primary Network at or after the `startTime`, and the validator must finish validating the Primary Network at or before the `endTime`.

`weight`

This is the validator’s sampling weight for consensus. If the validator’s weight is 1 and the cumulative weight of all validators in the subnet is 100, then this validator will be included in about 1 in every 100 samples during consensus.

`changeAddr`

If this address will receive any changes resulting from this transaction. You can leave this field blank; if you do, money will be delivered to one of your user's addresses.

`username` and `password`

The username and password of the user who pays the transaction charge are included in these parameters. To add a validator to this Subnet, this user must have access to a sufficient number of the Subnet's control keys.

To utilise as the values of `startTime` and `endTime` we use the shell programme 'date' to compute the Unix time 10 minutes and 30 days in the future, respectively. \(Note: If you’re on a Mac, replace `$(date` with `$(gdate`. If you don’t have `gdate` installed, do `brew install coreutils`.\)

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addSubnetValidator",
    "params": {
        "nodeID":"NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
        "subnetID":"nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr",
        "startTime":'$(date --date="10 minutes" +%s)',
        "endTime":'$(date --date="30 days" +%s)',
        "weight":1,
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u",
        "username":"USERNAME",
        "password":"PASSWORD"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response has the transaction ID, as well as the address the change went to.

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "txID": "2exafyvRNSE5ehwjhafBVt6CTntot7DFjsZNcZ54GSxBbVLcCm",
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u"
    },
    "id": 1
}
```

We can check the status of the transaction by calling [`platform.getTxStatus`](https://docs.avax.network/build/avalanchego-apis/issuing-api-calls):

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getTxStatus",
    "params": {
        "txID":"2exafyvRNSE5ehwjhafBVt6CTntot7DFjsZNcZ54GSxBbVLcCm"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The status should be `Committed`, meaning the transaction was successful. We can call [`platform.getPendingValidators`](https://docs.avax.network/build/tutorials/nodes-and-staking/add-a-validator) and see that the node is now in the pending validator set for the Primary Network. This time, we specify the subnet ID:

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {"subnetID":"nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr"},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The node we just added should be included in the response:

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "validators": [
            {
                "nodeID": "NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
                "startTime":1584042912,
                "endTime":1584121156,
                "weight": "1"
            }
        ]
    },
    "id": 1
}
```

This node will begin verifying this Subnet when the time reaches `1584042912`. This node will finish verifying this Subnet when it reaches `1584121156`.

### Whitelisting the Subnet

Now that the node has been added as a validator of the subnet, let’s add it to the whitelist of subnets. The whitelist prevents the node from validating a subnet unintentionally.

restart the node and add the parameter to whitelist the subnet `--whitelisted-subnets` with a comma-separated list of subnets to whitelist.

The full command is:

`./build/avalanchego --whitelisted-subnets=nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr`

# About the author

[Devesh jain] (https://community.figment.io/u/deveshjain08)

# References

https://docs.avax.network/build/tutorials/nodes-and-staking/add-a-validator
https://docs.avax.network/build/avalanchego-apis/issuing-api-calls
https://docs.avax.network/build/tutorials/nodes-and-staking/add-a-validator#:~:text=Add%20a%20validator%20with%20Avalanche%20Wallet&text=Open%20the%20wallet%2C%20and%20go,Choose%20Add%20Validator%20.&text=Fill%20out%20the%20staking%20parameters,explained%20in%20more%20detail%20below.
https://docs.avax.network/build/tutorials/platform/create-a-subnet
https://docs.avax.network/build/avalanchego-apis/issuing-api-calls
https://docs.avax.network/learn/platform-overview/staking
