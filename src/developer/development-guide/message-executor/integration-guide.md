# Integration Guide

## Overview

This tutorial shows how to run the [cross-chain batch transfer app](../contract-examples/batch-transfer.md) with your own executor. As an overview, our app does the following things:

1. User calls [`batchTransfer`](https://github.com/celer-network/sgn-v2-contracts/blob/e728cfc477/contracts/message/apps/examples/BatchTransfer.sol#L48) on the source chain Goerli Testnet, which triggers the sending of tokens along with a message through the Celer IM infrastructure
2. Executor polls Celer's SGN and submits SGN-signed messages to the `MessageBus` contract on the destination chain BSC Testnet.
3. `BatchTransfer` contract on BSC Testnet receives the message and distributes the fund to the receivers specified in the message

Following this tutorial, you will need to deploy `BatchTransfer` contracts on Goerli and BSC Testnets, and an Executor node for this test dApp.

### Prerequisites

1. Solidity Knowledge
2. Wallet
3. Node.js 12 installed
4. Typescript installed
5. Experience with basic Unix commands

## Contract

This tutorial uses [Hardhat](https://hardhat.org/) to deploy and verify the contract.

### Preparation

#### Download message-app-examples

```
git clone https://github.com/celer-network/message-app-examples.git
```

#### Get the dependencies

```
cd message-app-examples
yarn install
```

### Deploy the Contract

We are going to deploy [MsgExampleBasic](https://im-docs.celer.network/developer/development-guide/contract-examples/hello-world) on goerli and bsc testnet.&#x20;

Modify the config file `.env`. Set up your account's private key at `DEFAULT_PRIVATE_KEY`.

Deploy the contracts and remember to record the addresses of the deployed contracts as we will need them in the next step

```shell
# set MESSAGE_BUS_ADDR in .env to 0xF25170F86E4291a99a9A560032Fe9948b8BcFBB2
npx hardhat deploy --network goerli --tags MsgExampleBasic
# reset MESSAGE_BUS_ADDR to 0xAd204986D6cB67A5Bc76a3CB8974823F43Cb9AAA
npx hardhat deploy --network bscTest --tags MsgExampleBasic
```

### Verify the Contracts

You should now have the contract addresses on both networks.&#x20;

Now run the `hardhat verify` tasks. Note the last param is our contract's constructor param used when deploying the contract, which is the address of the message bus.

```shell
npx hardhat verify --network goerli <your-deployed-address-on-goerli> 0xF25170F86E4291a99a9A560032Fe9948b8BcFBB2
npx hardhat verify --network bscTest <your-deployed-address-on-bsc> 0xAd204986D6cB67A5Bc76a3CB8974823F43Cb9AAA
```

Woot, that's quite some work, if everything went right, you should be able to see your contracts on [GoerliScan](https://goerli.etherscan.io/) and [BscScan](https://testnet.bscscan.com/). Now we are just one component short of making an inter-chain app. Let's look into how to deploy the executor in the next section.

## Executor

In this section, we will learn what the executor is and how it should be configured and deployed

The executor is a simple program:  it polls SGN for available messages sent by the `BatchTransfer` contract and calls `MessageBus` on the destination chain which in turn calls our BatchTransfer's `executeMessageWithTransfer` on the destination chain.

> Note: "available messages" are messages that&#x20;
>
> 1. have been verified by enough SGN validators
> 2. have their corresponding token transfer verified

In addition, the executor also doesn't submit the message until the transfer associated with the message is executed on-chain.

Now let's start deploying the executor for our app.

### Preparation

Let's create a home folder for the executor first, this is where the config files will live

```shell
mkdir ~/.executor
```

Download the executor binary from [this repo](https://github.com/celer-network/sgn-v2-networks/tree/main/binaries), or use `curl`

```bash
# Linux amd64
curl -L https://github.com/celer-network/sgn-v2-networks/raw/main/binaries/executor-<latest-version>-linux-amd64.tar.gz -o executor.tar.gz
# Linux arm64
curl -L https://github.com/celer-network/sgn-v2-networks/raw/main/binaries/executor-<latest-version>-linux-arm64.tar.gz -o executor.tar.gz
# MacOS Intel chip
curl -L https://github.com/celer-network/sgn-v2-networks/raw/main/binaries/executor-<latest-version>-darwin-amd64.tar.gz -o executor.tar.gz
# MacOS Apple chip
curl -L https://github.com/celer-network/sgn-v2-networks/raw/main/binaries/executor-<latest-version>-darwin-arm64.tar.gz -o executor.tar.gz
```

Unzip it and move it to a directory on `$PATH`. We will use `/usr/local/bin`

```shell
tar -xvf executor.tar.gz && rm executor.tar.gz
mv executor-* /usr/local/bin/executor # may need sudo, or change this to your preferred location
```

Make sure the binary runs

```shell
# shell
executor start
# output
2022-04-20 17:19:23.126 |INFO | root.go:49: Reading executor configs
2022-04-20 17:19:23.127 |INFO | start.go:48: Starting executor
...
```

It won't actually run since we haven't setup any configs yet, but good to know it at least starts. If it doesn't, make sure you got the right distribution for your system arch.

### Database Setup

Since the executor monitors on-chain events and keeps track of message execution, we'll need a database. In theory, the executor supports any databases that support any Postgresql dialect database, but it's only tested with CockroachDB for now

#### Installation

You can visit their [website](https://www.cockroachlabs.com/docs/v21.2/install-cockroachdb-linux) for more detailed instructions. Below is an example for running CockroachDB on macOS via Homebrew

```shell
brew install cockroachdb/tap/cockroach
```

#### Start DB Instance

Start a single node instance in the background

```shell
cockroach start-single-node --store="$HOME/.crdb-node0" --listen-addr=localhost:26257 --http-addr=localhost:38080 --background --insecure
```

Test connection

```shell
cockroach sql --insecure
```

If you see this prompt then everything is right

```shell
# Welcome to the CockroachDB SQL shell.
# All statements must be terminated by a semicolon.
# To exit, type: \q.
#
# Server version: CockroachDB CCL v21.2.3 (x86_64-apple-darwin19, built 2021/12/14 15:26:20, go1.16.6) (same version as client)
# Cluster ID: 67881086-b544-4159-803f-2f7b952e1436
#
# Enter \? for a brief introduction.
#
root@:26257/defaultdb>
```

I know that's a lot of steps ... but thankfully that's all for the database. We are almost done here, just a little more configs, then we are off!

### Configurations

The config is simple, you only need two config files and an ETH keystore file.

First, let's create the folders and files in the executor home

```shell
.executor/
  - config/
      - executor.toml
      - cbridge.toml
  - eth-ks/
      - signer.json
```

Now we have the files in place, let's take a look at each individual file and what they do.

#### signer.json

Since the job of the executor is to submit messages on-chain, a signer keystore is required. Eventually, you may want to delegate the gas cost of the transactions the executor makes to your users, but that's outside of the scope of this tutorial. We will discuss this topic in later chapters.

#### executor.toml

This config file houses information about app contract, connectivity, and keystore location. A standard `executor.toml` looks like this. Remember to fill in the contract addresses and the keystore passphrase.

```toml
# since we don't want the executor to execute messages that are not sent by our
# BatchTransfer contract, the following items are added to filter only
# the ones we care about
[[service]]
# Fully qualified absolute path only, "~" would not work
signer_keystore = "/Users/patrickmao/.executor/eth-ks/signer.json"
signer_passphrase = "<your-keystore-passphrase>"
[[service.contracts]]
chain_id = 5 # Goerli
address = "<BatchTransfer-address>"
allow_sender_groups = ["batch-transfer"]
[[service.contracts]]
chain_id = 97 # Bsc testnet
address = "<BatchTransfer-address>"
allow_sender_groups = ["batch-transfer"]
[[service.contract_sender_groups]]
# the name/ID of the group. service.contracts refer to a sender group in allow_sender_groups
name = "batch-transfer" 
allow = [
  # allow and execute messages originated from <BatchTransfer-address> on chain 1
  { chain_id = 5, address = "<BatchTransfer-address>" },
  # allow and execute messages originated from <BatchTransfer-address> on chain 56
  { chain_id = 97, address = "<BatchTransfer-address>" },
]

[sgnd]
# SGN testnet node0 grpc. executor reads available messages from this endpoint
sgn_grpc = "cbridge-v2-test.celer.network:9094" 
# SGN testnet gateway grpc. all tx operations to the SGN is delegated through it
gateway_grpc = "cbridge-v2-test.celer.network:9094" 

[db]
url = "localhost:26257"
```

#### cbridge.toml

Executor relies on multiple on-chain events to do its job. This config file is where we configure on-chain event monitoring behaviors. The only things we need to care about for now is the address of the contracts and RPC endpoint URLs

```toml
[[multichain]]
chainID = 5
name = "Goerli"
gateway = "<your-goerli-rpc>" # fill in your Goerli rpc provider url
# cBridge (liquidity bridge) contract address. Executor relies on events from this
# contract to double check and make sure funds are transfered to the destination
# before it attempts messages on the destination chain
cbridge = "<copy-addr-from-'Contract Addresses & RPC Info'>"
# MessageBus contract address. Executor relies this to keep a message execution
# history (just so you can debug or help out angry customers).
msgbus = "<copy-addr-from-'Contract Addresses & RPC Info'>"
blkinterval = 15 # polling interval
blkdelay = 5 # how many blocks confirmations are required
maxblkdelta = 5000 # max number of blocks per poll request

[[multichain]]
chainID = 97
name = "BSC Testnet"
gateway = "https://data-seed-prebsc-2-s3.binance.org:8545/"
cbridge = "<copy-addr-from-'Contract Addresses & RPC Info'>"
msgbus = "<copy-addr-from-'Contract Addresses & RPC Info'>"
blkinterval = 3
blkdelay = 8
maxblkdelta = 5000
# on some EVM chains the gas estimation can be off. the below fields 
# are added to make up for the inconsistancies.
addgasgwei = 2 # add 2 gwei to gas price
addgasestimateratio = 0.3 # multiply gas limit by this ratio
```

### Running the Executor

Now with the configs and database out of the way, running the executor is as simple as a line of command (we'll discuss more reliable deployment methods in [Integration Tutorial: Advanced](integration-guide-advanced.md))

```shell
executor start --loglevel debug --home $HOME/.executor
```

> Sometimes executor start might fail because of failures to dial either SGN node or SGN gateway gRPC. It's probably because we are deploying something. Just wait a while and it'll most likely resolve.

That's it, the entire app stack is fully functional now. We've come a long way, and now is the moment of truth, will it work or not?&#x20;

## Testing the App

For testing, we are using test CELR on Goerli. Please add it to your wallet `0x5d3c0f4ca5ee99f8e8f59ff9a5fab04f6a7e007f`

#### Send the Request

Now we are ready to call our MsgExampleBasic contract on Goerli to initiate the whole cross-chain batch transfer process.

Note: the first param payable amount is the fee for this cross-chain transaction, we are omitting this for now.

```shell
_dstContract <MsgExampleBasic-address-on-bsc>
_dstChainId 97
_message 0x1234
```

After calling the contract, it may take around 30 \~ 120 seconds or so for SGN to monitor, verify and sign the transfer and the message. The executor will automatically pick up the message. The logs should look like this

```shell
│2022-04-27 01:19:42.781 |INFO | executor.go:542: executed xferMsg (id f66aec9401cbc77b525eafccaec49b6f4fe1a0af10c2c26858ee6d47c7628ef0): txhash 2c7fdaa4052af312119553bbae56d47322b981859254fb013afb29a3a40e19e2                                                                                  │
```

## Next: Advanced Topics

To make the BatchTransfer App more production-ready, there are some advanced functionalities that need to be added, which are covered in the [Integration Guide: Advanced](integration-guide-advanced.md).
