# Celer IM Design Patterns

In this section, we introduce the architecture of Celer IM via a step-by-step walkthrough of common application design patterns.&#x20;

### Cross-chain logic execution with accompanying fund transfer <a href="#cross-chain-logic-execution-with-accompanying-fund-transfer" id="cross-chain-logic-execution-with-accompanying-fund-transfer"></a>

For many inter-chain-native applications, the core flow often involves the process of sending funds to one or more chains and using those bridged funds to _do something_ on the destination chain(s). In fact, the DEX demo given above uses this exact pattern. Links to the demo code will be provided throughout this walkthrough.&#x20;

![](https://lh6.googleusercontent.com/CVTliSOnOQigmDf-dcU-_Y0nqW7ylXfcfYbzoYYwZt-SB0iV7wL1vMaqZxKK3sQr75qXG1pFxqeTpgds63NUL-GSkvRcrBy_yP8H8bIBYCDZGSrRXYLY_IVm66OqkdjrcvSS43AH)

While the above flow diagram seems complicated, we want to highlight that most of this flow is handled by Celer IM, and that developers will only need to work with two simple functions in the framework’s [application template](https://github.com/celer-network/sgn-v2-contracts/tree/main/contracts/message/framework).&#x20;

**Step 1: User initiates a transaction to dApp**

Instead of interfacing directly with the existing dApp smart contracts, a user now interacts (mark A) with a new dApp Plug-in contract to express their intention of cross-chain logic execution. This dApp plug-in becomes part of the overall dApp business logic and may interact with existing smart contracts on the source chain. This is usually the only transaction that a user sends to interact with this inter-chain dApp.&#x20;

In the DEX example shown, the [transferWithSwap](https://github.com/celer-network/sgn-v2-contracts/blob/1c65d5538ff8509c7e2626bb1a857683db775231/contracts/message/apps/examples/TransferSwap.sol#L104) function serves as this entry point that allows a user to express the intention of “swap Token A to Token B on Chain X and use the resulting Token B to swap for Token C on Chain Y”.

Of course, users usually do not manually specify these intentions. dApps using this framework are expected to compose higher-level user intent to these kinds of function calls.&#x20;

**Step 2: dApp Plug-in sends a message and associated cross-chain fund transfer**

After completing the necessary actions on the source chain, the dApp Plug-in sends the resulting funds and the associated message across to the destination chain (marked B, C). The message specifies the action that needs to be carried out on the destination chain. In the [DEX’s example](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/apps/examples/TransferSwap.sol), it is “swap the bridged token B to token C and give token C to the user”. The message and the fund transfer are automatically associated together by simply calling [sendMessageWithTransfer](https://github.com/celer-network/sgn-v2-contracts/blob/1c65d5538ff8509c7e2626bb1a857683db775231/contracts/message/apps/examples/TransferSwap.sol#L104). The message is then sent to the [Message Bus contract](https://github.com/celer-network/sgn-v2-contracts/tree/main/contracts/message/messagebus), and the fund transfer is sent via an asset bridge, in this case, [cBridge](http://cbridge.celer.network/).&#x20;

**Note:** Celer IM can utilize other asset bridges in this application pattern, while cBridge is just the first asset bridge that is supported.&#x20;

**Step 3: State Guardian Network (SGN) routes the message and cross-chain fund transfer**&#x20;

To understand this step, we must first introduce a core component in Celer IM: the State Guardian Network (SGN). The SGN is a Proof-of-Stake (PoS) blockchain built on Tendermint that serves as the **message router** between different blockchains. Node providers have to stake CELR tokens to join the consensus process of the SGN as a validator. The SGN uses the same security mechanisms as L1 blockchains like the Cosmos and Polygon PoS chains. The SGN’s CELR staking and slashing mechanisms are all implemented on Ethereum L1 smart contracts.&#x20;

The SGN validator nodes are continuously monitoring the transactions happening on all of the connected chains. When a transaction triggers a [cross-chain message event](https://github.com/celer-network/sgn-v2-contracts/blob/1c65d5538ff8509c7e2626bb1a857683db775231/contracts/message/messagebus/MessageBusSender.sol#L15-L27) in the [Message Bus](https://github.com/celer-network/sgn-v2-contracts/tree/main/contracts/message/messagebus) contract (marked D), validators will first reach a consensus on the existence of such message and concurrently generate a stake-weighed multi-signature attestation. This attestation is then stored on the SGN chain and waits to be relayed to the destination via an Executor subscribing to the message (marked H).&#x20;

For the cross-chain asset transfer, the cBridge contract can be seen as a specialized message bus with built-in optimizations for this purpose. A similar consensus and attestation process takes place (marked E). Instead of relaying this built-in fund transfer attestation to an off-chain Executor, the SGN validators themselves send the on-chain transaction to the cBridge contract (marked F) and trigger the fund transfer to the destination chain’s dApp Plug-in contract (marked G). Again, Celer IM can be connected to **any asset bridge** but starts with Celer’s [cBridge](http://cbridge.celer.network/) set as the default.&#x20;

**Step 4: Executor performs cross-chain application logic**&#x20;

The Executor’s task is to read the stake-weighted multi-signature attestation from the SGN blockchain and simply relay it to the Message Bus on the destination chain (marked I). An Executor can be run by anyone for any application as the functionality is simply relaying the message. Of course, dApps are expected to take Executor incentives into consideration as it is the entity that sends out the transaction and pays the gas fee on the destination chain.&#x20;

The functionality of the MessageBus is to check the validity of the attested message and verify that the associated payment has been received by the dApp Plug-in (mark J). After that, the message (logic execution instruction) is delivered to the dApp Plug-in contract, which hosts the dApp’s inter-chain business logic on the destination chain (marked K).&#x20;

The dApp Plug-in only needs to implement the [executeMessageWithTransfer](https://github.com/celer-network/sgn-v2-contracts/blob/1c65d5538ff8509c7e2626bb1a857683db775231/contracts/message/framework/MessageReceiverApp.sol#L47) interface. In the DEX example, [this function](https://github.com/celer-network/sgn-v2-contracts/blob/1c65d5538f/contracts/message/apps/examples/TransferSwap.sol#L249) will execute the “Token B to Token C swap” on the destination chain.

### Cross-chain logic execution without fund transfer <a href="#cross-chain-logic-execution-without-fund-transfer" id="cross-chain-logic-execution-without-fund-transfer"></a>

Many applications only need to send cross-chain messages or logic execution instructions without fund transfer. In the NFT marketplace for example, if a user participates in an auction that takes place on a different chain, they will only need to lock up their funds without actually transferring them to the destination chain in order to place a bid. It is only after they win the auction that a fund transfer will be required.&#x20;

![](https://lh5.googleusercontent.com/bSDQ8rOhrFxfsrzAY3oRgz0FvVhyWgj3Fy3OG-agY9zuUXQeR6Rdj9fhBC-0KvACab2UjqGwxcfHA9kv-GevdXug5r7odCPTnTn4nTXmB6FiCLir8889g4VUECSPce11Jpo6HHUz)

The flow for this would just be a simplified version of the first pattern. The dApp Plugin would only need to implement the logic to call [sendMessage](https://github.com/celer-network/sgn-v2-contracts/blob/1c65d5538f/contracts/message/framework/MessageSenderApp.sol#L28) on the source chain and then implement the [executeMessage](https://github.com/celer-network/sgn-v2-contracts/blob/1c65d5538f/contracts/message/framework/MessageReceiverApp.sol#L21) function on the destination chain’s dApp Plug-in contract.&#x20;

### Failure Handling <a href="#failure-handling" id="failure-handling"></a>

Due to the asynchronous nature of the above inter-chain messaging patterns, failure handling should be considered as part of the application logic. In these application patterns, failures can happen in the following three steps, and they should each be handled accordingly:

1. Source chain dApp logic execution failure. This is not related to Celer IM and should be handled by the dApp business logic itself (e.g. deadline exceeded for a DEX swap).&#x20;
2. Fund transfer failed in the asset bridge. The source chain dApp will be notified via a common interface and handle the refunded asset transfer by either retrying the fund transfer or sending it back to the user.&#x20;
3. Destination chain dApp logic execution failure. When a user’s fund reaches the destination chain, the dApp logic execution can still fail at that point. The dApp developers should prepare for this and should implement fallback functions in order to handle such a failure. A common way to handle such a failure can be to stop the execution and send the funds to the user on the destination chain or transfer the funds back to the source chain, but it is entirely up to the dApp developer to implement the specific logic of the fallback functions.&#x20;
