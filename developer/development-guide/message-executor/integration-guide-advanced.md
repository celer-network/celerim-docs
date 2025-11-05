# Integration Guide: Advanced

In the [basic guide](integration-guide.md) from the previous section, we walked through a minimal implementation of a cross-chain app. In this guide, we are going to discuss the following topics to make the [BatchTransfer](https://github.com/celer-network/sgn-v2-contracts/blob/ed81198f85/contracts/message/apps/examples/BatchTransfer.sol) inter-chain dApp more production ready. The goal is to get you on board with the patterns involved in deploying a robust service on top of the Celer IM infrastructure.

* Enhance security through executor's "sender groups" config
* Dealing with bridge failures (refunds)
* Chaining messages
* Configuring a retry back-off strategy

### Enhance security through executor's "sender groups" config

Under the current IM architecture, any contract can send messages to any other contracts. This means that a malicious party can forge messages that conform to your contract's message data type, send them to your contract, and exhaust your executor's gas fund. Thus, in production, it is important that the executor checks where a message originated from. Sender groups are designed just for that.

If you have experience with cloud services such as AWS, your might recognize that a sender group is pretty much a "security group".

An example sender group looks like this

```toml
[[service.contract_sender_groups]]
# the name/ID of the group. service.contracts refer to a sender group in allow_sender_groups
name = "your-sender-group" 
allow = [
  # allow and execute messages originated from <app-contract-address> on chain 1
  { chain_id = 5, address = "<app-contract-address>" },
  # allow and execute messages originated from <app-contract-address> on chain 56
  { chain_id = 97, address = "<app-contract-address>" },
]
```

After defining the security groups, we need to mount it to individual contract configs

```toml
[[service]]
[[service.contracts]]
chain_id = 5
address = "<app-contract-address>"
allow_sender_groups = ["your-sender-group"]
[[service.contracts]]
chain_id = 97
address = "<app-contract-address>"
allow_sender_groups = ["your-sender-group"]
```

### Dealing with Bridge Failures

#### Contract Changes

It is possible that bridging would fail when the user calls `batchTransfer` due to high slippage, not enough liquidity, etc. In These cases, the executor automatically prepares a refund and executes it on the source chain. In order for this to work, the BatchTransfer contract on the source chain needs to implement [`executeMessageWithTransferRefund`](https://github.com/celer-network/sgn-v2-contracts/blob/ed81198f85/contracts/message/apps/examples/BatchTransfer.sol#L88-L99).

Note that this function is called with the original \_message we encoded and sent out. And the funds are guaranteed to arrive before it is called.

#### Executor Changes

The executor has an option that needs to be explicitly turned on to enable auto refund for bridge failures.&#x20;

```toml
# executor.toml
[executor]
enable_auto_refund = true
```

### Chaining Messages

You may want to "chain" or "nest" a message in `executeMessageWithTransfer` on the destination chain. Since the `executeMessageWithTransfer` interface is payable, this usage is supported.

In the BatchTransfer contract, [a "receipt" message is chained inside](https://github.com/celer-network/sgn-v2-contracts/blob/ed81198f85/contracts/message/apps/examples/BatchTransfer.sol#L138-L140).&#x20;

Now we need to configure the executor to add a payable value when calling `executeMessageWithTransfer` to cover the fee introduced by chaining the additional message.

```toml
[service]
[[service.contracts]]
chain_id = 5                                           # Goerli
address = "0x09E4534B11D400BFcd2026b69E399763CeAfB42D"
add_payable_value_for_execution = 20000000000 # <-- add this line, amount in wei
[[service.contracts]]
chain_id = 97                                          # Bsc testnet
address = "0x570F9c2f224b002d75F287f5430Bc9598E850E13"
add_payable_value_for_execution = 20000000000 # <-- add this line, amount in wei
```

But how do we know how much fee is needed? This is the tricky part since we can only estimate the amount of fee our `sendMessage`call is incurring. Let's take a look at how the message fee is calculated in the MessageBus contract:&#x20;

```solidity
uint256 public feeBase;
uint256 public feePerByte;

function calcFee(bytes calldata _message) public view returns (uint256) {
    return feeBase + _message.length * feePerByte;
}
```

You can query the MessageBus contract on a chain for these parameters. If you use `abi.encode` to encode your message, the message length is likely fixed. If you happen to have variable length fields in your message, you should add a safe margin to the `add_payable_value_for_execution` to reduce the chance of having message execution reverted.
